# Java Filter Examples for ACTIVITY-TRACKING System
## Using JSF, PrimeFaces Framework

### 1. Activity Tracking Filter (Java Servlet Filter)

#### ActivityTrackingFilter.java
```java
package com.company.activitytracking.filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.util.Date;
import java.util.logging.Logger;

import com.company.activitytracking.service.ActivityLogService;
import com.company.activitytracking.model.ActivityLog;
import com.company.activitytracking.util.IPUtils;
import com.company.activitytracking.util.UserAgentUtils;

/**
 * Activity Tracking Filter - ติดตามกิจกรรมทุกครั้งที่มี HTTP Request
 * ใช้งานร่วมกับ JSF และ PrimeFaces
 */
@WebFilter(filterName = "ActivityTrackingFilter", urlPatterns = {"*.xhtml", "*.jsf"})
public class ActivityTrackingFilter implements Filter {
    
    private static final Logger logger = Logger.getLogger(ActivityTrackingFilter.class.getName());
    private ActivityLogService activityLogService;
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // Initialize ActivityLogService (can use CDI injection in real application)
        activityLogService = new ActivityLogService();
        logger.info("ActivityTrackingFilter initialized");
    }
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        HttpSession session = httpRequest.getSession(false);
        
        // Skip tracking for static resources
        String requestURI = httpRequest.getRequestURI();
        if (isStaticResource(requestURI)) {
            chain.doFilter(request, response);
            return;
        }
        
        // Track request start time
        long startTime = System.currentTimeMillis();
        
        try {
            // Continue with the request
            chain.doFilter(request, response);
            
            // Log successful request
            logActivity(httpRequest, httpResponse, startTime, null);
            
        } catch (Exception e) {
            // Log failed request
            logActivity(httpRequest, httpResponse, startTime, e);
            throw e;
        }
    }
    
    private void logActivity(HttpServletRequest request, HttpServletResponse response, 
                           long startTime, Exception error) {
        try {
            HttpSession session = request.getSession(false);
            if (session == null) return; // No session, skip logging
            
            String userId = (String) session.getAttribute("userId");
            if (userId == null) return; // Not logged in, skip logging
            
            ActivityLog log = new ActivityLog();
            log.setUserId(userId);
            log.setSessionId(session.getId());
            log.setTimestamp(new Date());
            log.setRequestURI(request.getRequestURI());
            log.setHttpMethod(request.getMethod());
            log.setIpAddress(IPUtils.getMaskedClientIP(request));
            log.setUserAgent(request.getHeader("User-Agent"));
            log.setDeviceInfo(UserAgentUtils.getDeviceInfo(request.getHeader("User-Agent")));
            log.setResponseTime(System.currentTimeMillis() - startTime);
            log.setResponseStatus(response.getStatus());
            
            // Determine activity type based on URL
            log.setActivityType(determineActivityType(request.getRequestURI()));
            
            // Log error if present
            if (error != null) {
                log.setErrorMessage(error.getMessage());
                log.setStatus("ERROR");
            } else {
                log.setStatus("SUCCESS");
            }
            
            // Async logging to avoid performance impact
            activityLogService.logActivityAsync(log);
            
        } catch (Exception e) {
            logger.severe("Failed to log activity: " + e.getMessage());
        }
    }
    
    private String determineActivityType(String requestURI) {
        if (requestURI.contains("/login")) return "AUTHENTICATION";
        if (requestURI.contains("/document")) return "DOCUMENT_ACCESS";
        if (requestURI.contains("/approval")) return "APPROVAL_PROCESS";
        if (requestURI.contains("/module")) return "MODULE_USAGE";
        return "PAGE_ACCESS";
    }
    
    private boolean isStaticResource(String requestURI) {
        return requestURI.contains("/resources/") || 
               requestURI.endsWith(".css") || 
               requestURI.endsWith(".js") || 
               requestURI.endsWith(".png") || 
               requestURI.endsWith(".jpg") || 
               requestURI.endsWith(".gif") ||
               requestURI.endsWith(".ico");
    }
    
    @Override
    public void destroy() {
        logger.info("ActivityTrackingFilter destroyed");
    }
}
```

---

### 2. Document Access Filter (Specific for Document Module)

#### DocumentAccessFilter.java
```java
package com.company.activitytracking.filter;

import javax.servlet.*;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.io.IOException;
import java.util.Date;

import com.company.activitytracking.service.DocumentAccessLogService;
import com.company.activitytracking.model.DocumentAccessLog;

/**
 * Document Access Filter - ติดตามการเข้าถึงเอกสารเฉพาะ
 */
@WebFilter(filterName = "DocumentAccessFilter", urlPatterns = {"/document/*", "/files/*"})
public class DocumentAccessFilter implements Filter {
    
    private DocumentAccessLogService docLogService;
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        docLogService = new DocumentAccessLogService();
    }
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpSession session = httpRequest.getSession(false);
        
        // Extract document information from request
        String documentId = extractDocumentId(httpRequest);
        String action = extractAction(httpRequest);
        
        if (documentId != null && session != null) {
            String userId = (String) session.getAttribute("userId");
            
            if (userId != null) {
                // Log document access
                DocumentAccessLog docLog = new DocumentAccessLog();
                docLog.setUserId(userId);
                docLog.setDocumentId(documentId);
                docLog.setActionType(action);
                docLog.setTimestamp(new Date());
                docLog.setSessionId(session.getId());
                docLog.setModuleName(determineModule(httpRequest));
                
                // Track access start time for duration calculation
                httpRequest.setAttribute("docAccessStartTime", System.currentTimeMillis());
                httpRequest.setAttribute("documentLog", docLog);
            }
        }
        
        chain.doFilter(request, response);
        
        // Log completion (for VIEW duration tracking)
        logDocumentAccessCompletion(httpRequest);
    }
    
    private void logDocumentAccessCompletion(HttpServletRequest request) {
        DocumentAccessLog docLog = (DocumentAccessLog) request.getAttribute("documentLog");
        if (docLog != null && "VIEW".equals(docLog.getActionType())) {
            Long startTime = (Long) request.getAttribute("docAccessStartTime");
            if (startTime != null) {
                long duration = (System.currentTimeMillis() - startTime) / 1000; // seconds
                docLog.setAccessDuration(duration);
            }
            docLogService.logDocumentAccessAsync(docLog);
        }
    }
    
    private String extractDocumentId(HttpServletRequest request) {
        // Extract from URL parameter or path
        String docId = request.getParameter("documentId");
        if (docId == null) {
            String pathInfo = request.getPathInfo();
            if (pathInfo != null && pathInfo.contains("/doc/")) {
                String[] parts = pathInfo.split("/");
                for (int i = 0; i < parts.length - 1; i++) {
                    if ("doc".equals(parts[i]) && i + 1 < parts.length) {
                        docId = parts[i + 1];
                        break;
                    }
                }
            }
        }
        return docId;
    }
    
    private String extractAction(HttpServletRequest request) {
        String method = request.getMethod();
        String uri = request.getRequestURI();
        
        if ("POST".equals(method) && uri.contains("/upload")) return "UPLOAD";
        if ("GET".equals(method) && uri.contains("/download")) return "DOWNLOAD";
        if ("POST".equals(method) && uri.contains("/share")) return "SHARE";
        if ("DELETE".equals(method)) return "DELETE";
        return "VIEW";
    }
    
    private String determineModule(HttpServletRequest request) {
        String referer = request.getHeader("Referer");
        if (referer != null) {
            if (referer.contains("/hr/")) return "HR_MODULE";
            if (referer.contains("/finance/")) return "FINANCE_MODULE";
            if (referer.contains("/audit/")) return "AUDIT_MODULE";
        }
        return "DOCUMENT_MODULE";
    }
    
    @Override
    public void destroy() {
        // Cleanup
    }
}
```

---

### 3. JSF Phase Listener for Module Tracking

#### ModuleTrackingPhaseListener.java
```java
package com.company.activitytracking.listener;

import javax.faces.context.FacesContext;
import javax.faces.event.PhaseEvent;
import javax.faces.event.PhaseId;
import javax.faces.event.PhaseListener;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;
import java.util.Date;

import com.company.activitytracking.service.ModuleUsageLogService;
import com.company.activitytracking.model.ModuleUsageLog;

/**
 * JSF Phase Listener สำหรับติดตาม Module Usage
 */
public class ModuleTrackingPhaseListener implements PhaseListener {
    
    private ModuleUsageLogService moduleLogService = new ModuleUsageLogService();
    
    @Override
    public void afterPhase(PhaseEvent event) {
        // Track after RESTORE_VIEW phase (page load)
        if (event.getPhaseId() == PhaseId.RESTORE_VIEW) {
            trackModuleUsage(event.getFacesContext());
        }
    }
    
    @Override
    public void beforePhase(PhaseEvent event) {
        // Track before RENDER_RESPONSE phase (page exit)
        if (event.getPhaseId() == PhaseId.RENDER_RESPONSE) {
            trackModuleExit(event.getFacesContext());
        }
    }
    
    private void trackModuleUsage(FacesContext facesContext) {
        HttpServletRequest request = (HttpServletRequest) facesContext.getExternalContext().getRequest();
        HttpSession session = request.getSession(false);
        
        if (session == null) return;
        
        String userId = (String) session.getAttribute("userId");
        if (userId == null) return;
        
        String currentModule = determineModule(request.getRequestURI());
        String previousModule = (String) session.getAttribute("currentModule");
        
        if (currentModule != null && !currentModule.equals(previousModule)) {
            // Log exit from previous module
            if (previousModule != null) {
                logModuleExit(userId, session.getId(), previousModule, currentModule);
            }
            
            // Log entry to current module
            logModuleEntry(userId, session.getId(), currentModule, previousModule);
            
            // Update session with current module
            session.setAttribute("currentModule", currentModule);
            session.setAttribute("moduleEntryTime", System.currentTimeMillis());
        }
    }
    
    private void trackModuleExit(FacesContext facesContext) {
        // This will be called on page unload or navigation
        // Additional logic can be added here for tracking page exit
    }
    
    private void logModuleEntry(String userId, String sessionId, String moduleName, String previousModule) {
        ModuleUsageLog log = new ModuleUsageLog();
        log.setUserId(userId);
        log.setSessionId(sessionId);
        log.setModuleName(moduleName);
        log.setActionType("ENTER");
        log.setTimestamp(new Date());
        log.setPreviousModule(previousModule);
        log.setNotes("Module entry tracked");
        
        moduleLogService.logModuleUsageAsync(log);
    }
    
    private void logModuleExit(String userId, String sessionId, String moduleName, String nextModule) {
        ModuleUsageLog log = new ModuleUsageLog();
        log.setUserId(userId);
        log.setSessionId(sessionId);
        log.setModuleName(moduleName);
        log.setActionType("EXIT");
        log.setTimestamp(new Date());
        log.setNextModule(nextModule);
        
        // Calculate duration if entry time is available
        HttpSession session = FacesContext.getCurrentInstance().getExternalContext().getSession(false);
        if (session != null) {
            Long entryTime = (Long) session.getAttribute("moduleEntryTime");
            if (entryTime != null) {
                long duration = (System.currentTimeMillis() - entryTime) / 1000; // seconds
                log.setDurationSeconds(duration);
            }
        }
        
        log.setNotes("Module exit tracked");
        moduleLogService.logModuleUsageAsync(log);
    }
    
    private String determineModule(String requestURI) {
        if (requestURI.contains("/hr/")) return "HR_MODULE";
        if (requestURI.contains("/finance/")) return "FINANCE_MODULE";
        if (requestURI.contains("/document/")) return "DOCUMENT_MODULE";
        if (requestURI.contains("/approval/")) return "APPROVAL_MODULE";
        if (requestURI.contains("/report/")) return "REPORT_CENTER";
        if (requestURI.contains("/audit/")) return "AUDIT_MODULE";
        if (requestURI.contains("/dashboard/")) return "DASHBOARD";
        return null;
    }
    
    @Override
    public PhaseId getPhaseId() {
        return PhaseId.ANY_PHASE;
    }
}
```

---

### 4. Configuration Files

#### faces-config.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<faces-config xmlns="http://xmlns.jcp.org/xml/ns/javaee"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
              http://xmlns.jcp.org/xml/ns/javaee/web-facesconfig_2_3.xsd"
              version="2.3">
    
    <!-- Register Phase Listener for Module Tracking -->
    <lifecycle>
        <phase-listener>
            com.company.activitytracking.listener.ModuleTrackingPhaseListener
        </phase-listener>
    </lifecycle>
    
    <!-- Navigation rules for modules -->
    <navigation-rule>
        <from-view-id>*</from-view-id>
        <navigation-case>
            <from-outcome>hr-module</from-outcome>
            <to-view-id>/hr/dashboard.xhtml</to-view-id>
            <redirect/>
        </navigation-case>
        <navigation-case>
            <from-outcome>finance-module</from-outcome>
            <to-view-id>/finance/dashboard.xhtml</to-view-id>
            <redirect/>
        </navigation-case>
        <navigation-case>
            <from-outcome>document-module</from-outcome>
            <to-view-id>/document/browser.xhtml</to-view-id>
            <redirect/>
        </navigation-case>
    </navigation-rule>
    
</faces-config>
```

#### web.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee 
         http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">
    
    <display-name>Activity Tracking System</display-name>
    
    <!-- JSF Configuration -->
    <context-param>
        <param-name>javax.faces.PROJECT_STAGE</param-name>
        <param-value>Production</param-value>
    </context-param>
    
    <context-param>
        <param-name>primefaces.THEME</param-name>
        <param-value>nova-light</param-value>
    </context-param>
    
    <!-- Faces Servlet -->
    <servlet>
        <servlet-name>Faces Servlet</servlet-name>
        <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>Faces Servlet</servlet-name>
        <url-pattern>*.xhtml</url-pattern>
    </servlet-mapping>
    
    <!-- Activity Tracking Filter is automatically registered via @WebFilter annotation -->
    
    <!-- Session timeout (30 minutes) -->
    <session-config>
        <session-timeout>30</session-timeout>
    </session-config>
    
    <!-- Security constraint for XHTML pages -->
    <security-constraint>
        <web-resource-collection>
            <web-resource-name>XHTML pages</web-resource-name>
            <url-pattern>*.xhtml</url-pattern>
        </web-resource-collection>
        <auth-constraint>
            <role-name>user</role-name>
        </auth-constraint>
    </security-constraint>
    
    <!-- Welcome file -->
    <welcome-file-list>
        <welcome-file>dashboard.xhtml</welcome-file>
    </welcome-file-list>
    
</web-app>
```

---

### 5. PrimeFaces Usage Examples

#### dashboard.xhtml (Main Dashboard with Activity Tracking)
```xhtml
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://xmlns.jcp.org/jsf/core"
      xmlns:p="http://primefaces.org/ui">

<h:head>
    <title>Activity Tracking Dashboard</title>
    <f:metadata>
        <!-- Track page access -->
        <f:event type="preRenderView" listener="#{dashboardBean.trackPageAccess}" />
    </f:metadata>
</h:head>

<h:body>
    <h:form id="dashboardForm">
        
        <!-- Navigation Menu with Activity Tracking -->
        <p:menubar>
            <p:menuitem value="Dashboard" url="/dashboard.xhtml" />
            <p:menuitem value="HR Module" action="#{navigationBean.navigateToHR}" 
                       update="@form" process="@this"/>
            <p:menuitem value="Finance Module" action="#{navigationBean.navigateToFinance}" 
                       update="@form" process="@this"/>
            <p:menuitem value="Documents" action="#{navigationBean.navigateToDocuments}" 
                       update="@form" process="@this"/>
            <p:menuitem value="Approvals" action="#{navigationBean.navigateToApprovals}" 
                       update="@form" process="@this"/>
        </p:menubar>
        
        <!-- Real-time Activity Panel -->
        <p:panel header="Real-time Activity" style="margin-top: 20px;">
            <p:dataTable id="realtimeTable" value="#{dashboardBean.realtimeActivities}" 
                        var="activity" paginator="true" rows="10"
                        paginatorTemplate="{CurrentPageReport} {FirstPageLink} {PreviousPageLink} {PageLinks} {NextPageLink} {LastPageLink}"
                        currentPageReportTemplate="Showing {startRecord} to {endRecord} of {totalRecords}">
                
                <p:column headerText="Time">
                    <h:outputText value="#{activity.timestamp}">
                        <f:convertDateTime pattern="HH:mm:ss" />
                    </h:outputText>
                </p:column>
                
                <p:column headerText="User">
                    <h:outputText value="#{activity.userId}" />
                </p:column>
                
                <p:column headerText="Activity">
                    <h:outputText value="#{activity.activityType}" />
                </p:column>
                
                <p:column headerText="Module">
                    <h:outputText value="#{activity.moduleName}" />
                </p:column>
                
                <p:column headerText="Status">
                    <p:tag value="#{activity.status}" 
                          severity="#{activity.status eq 'SUCCESS' ? 'success' : 'danger'}" />
                </p:column>
            </p:dataTable>
            
            <!-- Auto-refresh every 30 seconds -->
            <p:poll interval="30" update="realtimeTable" />
        </p:panel>
        
        <!-- Activity Charts -->
        <p:panel header="Activity Statistics" style="margin-top: 20px;">
            <div class="ui-g">
                <div class="ui-g-6">
                    <!-- Login Trends Chart -->
                    <p:chart type="line" model="#{dashboardBean.loginTrendsModel}" 
                            style="height:300px;width:100%" />
                </div>
                <div class="ui-g-6">
                    <!-- Module Usage Chart -->
                    <p:chart type="pie" model="#{dashboardBean.moduleUsageModel}" 
                            style="height:300px;width:100%" />
                </div>
            </div>
        </p:panel>
        
        <!-- Security Alerts -->
        <p:panel header="Security Alerts" style="margin-top: 20px;">
            <p:dataTable value="#{dashboardBean.securityAlerts}" var="alert"
                        emptyMessage="No security alerts">
                <p:column headerText="Time">
                    <h:outputText value="#{alert.timestamp}">
                        <f:convertDateTime pattern="dd/MM/yyyy HH:mm" />
                    </h:outputText>
                </p:column>
                
                <p:column headerText="Severity">
                    <p:tag value="#{alert.severity}" 
                          severity="#{alert.severity eq 'HIGH' ? 'danger' : 'warning'}" />
                </p:column>
                
                <p:column headerText="Description">
                    <h:outputText value="#{alert.description}" />
                </p:column>
                
                <p:column headerText="Action">
                    <p:commandButton value="View Details" 
                                   action="#{dashboardBean.viewAlertDetails(alert)}"
                                   update="alertDetailDialog" 
                                   oncomplete="PF('alertDialog').show()" />
                </p:column>
            </p:dataTable>
        </p:panel>
        
        <!-- Alert Detail Dialog -->
        <p:dialog id="alertDetailDialog" header="Alert Details" 
                 widgetVar="alertDialog" modal="true" width="600">
            <p:panelGrid columns="2" layout="grid">
                <h:outputText value="Alert ID:" />
                <h:outputText value="#{dashboardBean.selectedAlert.id}" />
                
                <h:outputText value="User ID:" />
                <h:outputText value="#{dashboardBean.selectedAlert.userId}" />
                
                <h:outputText value="IP Address:" />
                <h:outputText value="#{dashboardBean.selectedAlert.ipAddress}" />
                
                <h:outputText value="Description:" />
                <h:outputText value="#{dashboardBean.selectedAlert.description}" />
                
                <h:outputText value="Action Taken:" />
                <h:outputText value="#{dashboardBean.selectedAlert.actionTaken}" />
            </p:panelGrid>
            
            <f:facet name="footer">
                <p:commandButton value="Close" onclick="PF('alertDialog').hide()" />
            </f:facet>
        </p:dialog>
    </h:form>
    
    <!-- JavaScript for client-side activity tracking -->
    <script type="text/javascript">
        // Track user interactions
        document.addEventListener('click', function(e) {
            if (e.target.tagName === 'BUTTON' || e.target.tagName === 'A') {
                // Send async request to track button/link clicks
                trackUserInteraction('CLICK', e.target.id || e.target.className);
            }
        });
        
        function trackUserInteraction(action, element) {
            fetch('/activity-tracking/api/track', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    action: action,
                    element: element,
                    timestamp: new Date().toISOString()
                })
            });
        }
        
        // Track page unload
        window.addEventListener('beforeunload', function() {
            navigator.sendBeacon('/activity-tracking/api/page-exit', 
                JSON.stringify({timestamp: new Date().toISOString()}));
        });
    </script>
</h:body>
</html>
```

#### document-browser.xhtml (Document Module with File Access Tracking)
```xhtml
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://xmlns.jcp.org/jsf/html"
      xmlns:f="http://xmlns.jcp.org/jsf/core"
      xmlns:p="http://primefaces.org/ui">

<h:head>
    <title>Document Browser - Activity Tracking</title>
    <f:metadata>
        <f:event type="preRenderView" listener="#{documentBean.trackModuleAccess}" />
    </f:metadata>
</h:head>

<h:body>
    <h:form id="documentForm">
        
        <p:breadCrumb>
            <p:menuitem value="Dashboard" url="/dashboard.xhtml" />
            <p:menuitem value="Documents" />
        </p:breadCrumb>
        
        <!-- Document Browser -->
        <p:panel header="Document Browser" style="margin-top: 20px;">
            
            <!-- File Upload with Activity Tracking -->
            <p:fileUpload value="#{documentBean.uploadedFile}" 
                         mode="simple" 
                         label="Choose File"
                         uploadLabel="Upload"
                         cancelLabel="Cancel"
                         skinSimple="true"
                         listener="#{documentBean.uploadDocument}"
                         update="documentTable growl" />
            
            <!-- Document Table -->
            <p:dataTable id="documentTable" 
                        value="#{documentBean.documents}" 
                        var="doc" 
                        paginator="true" 
                        rows="20"
                        filteredValue="#{documentBean.filteredDocuments}"
                        globalFilterFunction="#{documentBean.globalFilter}">
                
                <f:facet name="header">
                    <div class="ui-g">
                        <div class="ui-g-6">
                            <h:outputText value="Documents" />
                        </div>
                        <div class="ui-g-6">
                            <p:inputText id="globalFilter" 
                                        onkeyup="PF('documentsTable').filter()" 
                                        placeholder="Search documents..." 
                                        style="width:100%" />
                        </div>
                    </div>
                </f:facet>
                
                <p:column headerText="Name" sortBy="#{doc.name}" filterBy="#{doc.name}">
                    <h:outputText value="#{doc.name}" />
                </p:column>
                
                <p:column headerText="Type" sortBy="#{doc.type}">
                    <h:outputText value="#{doc.type}" />
                </p:column>
                
                <p:column headerText="Size" sortBy="#{doc.size}">
                    <h:outputText value="#{doc.formattedSize}" />
                </p:column>
                
                <p:column headerText="Modified" sortBy="#{doc.lastModified}">
                    <h:outputText value="#{doc.lastModified}">
                        <f:convertDateTime pattern="dd/MM/yyyy HH:mm" />
                    </h:outputText>
                </p:column>
                
                <p:column headerText="Actions">
                    <!-- View Document (tracked by DocumentAccessFilter) -->
                    <p:commandButton value="View" 
                                   action="#{documentBean.viewDocument(doc)}"
                                   ajax="false"
                                   onclick="trackDocumentAction('VIEW', '#{doc.id}')" />
                    
                    <!-- Download Document (tracked by DocumentAccessFilter) -->
                    <p:commandButton value="Download" 
                                   action="#{documentBean.downloadDocument(doc)}"
                                   ajax="false"
                                   onclick="trackDocumentAction('DOWNLOAD', '#{doc.id}')" />
                    
                    <!-- Share Document -->
                    <p:commandButton value="Share" 
                                   update="shareDialog" 
                                   oncomplete="PF('shareDialog').show()"
                                   action="#{documentBean.prepareShare(doc)}" />
                    
                    <!-- Delete Document (for authorized users) -->
                    <p:commandButton value="Delete" 
                                   rendered="#{documentBean.canDelete(doc)}"
                                   action="#{documentBean.deleteDocument(doc)}"
                                   update="documentTable growl"
                                   onclick="trackDocumentAction('DELETE', '#{doc.id}')"
                                   oncomplete="if (!args.validationFailed) trackDocumentAction('DELETE_SUCCESS', '#{doc.id}')" >
                        <p:confirm header="Confirmation" 
                                  message="Are you sure you want to delete this document?" 
                                  icon="pi pi-exclamation-triangle" />
                    </p:commandButton>
                </p:column>
            </p:dataTable>
            
            <p:confirmDialog global="true" showEffect="fade" hideEffect="fade">
                <p:commandButton value="Yes" type="button" 
                               styleClass="ui-confirmdialog-yes" icon="pi pi-check" />
                <p:commandButton value="No" type="button" 
                               styleClass="ui-confirmdialog-no" icon="pi pi-times" />
            </p:confirmDialog>
        </p:panel>
        
        <!-- Document Share Dialog -->
        <p:dialog id="shareDialog" header="Share Document" 
                 widgetVar="shareDialog" modal="true" width="500">
            <p:panelGrid columns="2" layout="grid">
                <h:outputText value="Document:" />
                <h:outputText value="#{documentBean.selectedDocument.name}" />
                
                <h:outputText value="Share with users:" />
                <p:selectCheckboxMenu value="#{documentBean.selectedUsers}" 
                                     label="Select Users"
                                     filter="true" 
                                     filterMatchMode="contains">
                    <f:selectItems value="#{documentBean.availableUsers}" 
                                  var="user" 
                                  itemLabel="#{user.name}" 
                                  itemValue="#{user.id}" />
                </p:selectCheckboxMenu>
                
                <h:outputText value="Message:" />
                <p:inputTextarea value="#{documentBean.shareMessage}" 
                               rows="3" cols="40" />
            </p:panelGrid>
            
            <f:facet name="footer">
                <p:commandButton value="Share" 
                               action="#{documentBean.shareDocument}"
                               update="growl"
                               oncomplete="if (!args.validationFailed) { PF('shareDialog').hide(); trackDocumentAction('SHARE_SUCCESS', '#{documentBean.selectedDocument.id}'); }" />
                <p:commandButton value="Cancel" 
                               onclick="PF('shareDialog').hide()" />
            </f:facet>
        </p:dialog>
        
        <p:growl id="growl" showDetail="true" />
    </h:form>
    
    <!-- JavaScript for document action tracking -->
    <script type="text/javascript">
        function trackDocumentAction(action, documentId) {
            fetch('/activity-tracking/api/document-action', {
                method: 'POST',
                headers: {
                    'Content-Type': 'application/json',
                },
                body: JSON.stringify({
                    action: action,
                    documentId: documentId,
                    timestamp: new Date().toISOString(),
                    module: 'DOCUMENT_MODULE'
                })
            });
        }
        
        // Track document view duration
        let documentViewStartTime = null;
        
        function startTrackingViewDuration(documentId) {
            documentViewStartTime = new Date().getTime();
            window.currentDocumentId = documentId;
        }
        
        window.addEventListener('beforeunload', function() {
            if (documentViewStartTime && window.currentDocumentId) {
                let duration = (new Date().getTime() - documentViewStartTime) / 1000;
                navigator.sendBeacon('/activity-tracking/api/document-view-duration', 
                    JSON.stringify({
                        documentId: window.currentDocumentId,
                        duration: duration
                    }));
            }
        });
    </script>
</h:body>
</html>
```

---

### 6. Managed Bean Examples

#### DashboardBean.java
```java
package com.company.activitytracking.bean;

import javax.annotation.PostConstruct;
import javax.faces.bean.ManagedBean;
import javax.faces.bean.ViewScoped;
import javax.faces.context.FacesContext;
import javax.servlet.http.HttpServletRequest;
import java.io.Serializable;
import java.util.List;
import java.util.Date;

import org.primefaces.model.chart.ChartModel;
import org.primefaces.model.chart.LineChartModel;
import org.primefaces.model.chart.PieChartModel;

import com.company.activitytracking.model.ActivityLog;
import com.company.activitytracking.model.SecurityAlert;
import com.company.activitytracking.service.ActivityLogService;
import com.company.activitytracking.service.SecurityAlertService;

@ManagedBean
@ViewScoped
public class DashboardBean implements Serializable {
    
    private ActivityLogService activityLogService;
    private SecurityAlertService securityAlertService;
    
    private List<ActivityLog> realtimeActivities;
    private List<SecurityAlert> securityAlerts;
    private SecurityAlert selectedAlert;
    
    private LineChartModel loginTrendsModel;
    private PieChartModel moduleUsageModel;
    
    @PostConstruct
    public void init() {
        activityLogService = new ActivityLogService();
        securityAlertService = new SecurityAlertService();
        
        loadRealtimeActivities();
        loadSecurityAlerts();
        createCharts();
    }
    
    public void trackPageAccess() {
        // Track dashboard page access
        HttpServletRequest request = (HttpServletRequest) 
            FacesContext.getCurrentInstance().getExternalContext().getRequest();
        
        ActivityLog log = new ActivityLog();
        log.setUserId(getCurrentUserId());
        log.setTimestamp(new Date());
        log.setActivityType("DASHBOARD_ACCESS");
        log.setRequestURI(request.getRequestURI());
        log.setSessionId(request.getSession().getId());
        
        activityLogService.logActivityAsync(log);
    }
    
    private void loadRealtimeActivities() {
        // Load last 50 activities for real-time display
        realtimeActivities = activityLogService.getRecentActivities(50);
    }
    
    private void loadSecurityAlerts() {
        // Load active security alerts
        securityAlerts = securityAlertService.getActiveAlerts();
    }
    
    private void createCharts() {
        // Create login trends chart
        loginTrendsModel = new LineChartModel();
        // ... populate chart data
        
        // Create module usage pie chart
        moduleUsageModel = new PieChartModel();
        // ... populate chart data
    }
    
    public void viewAlertDetails(SecurityAlert alert) {
        this.selectedAlert = alert;
        // Mark alert as viewed
        securityAlertService.markAsViewed(alert.getId(), getCurrentUserId());
    }
    
    private String getCurrentUserId() {
        HttpServletRequest request = (HttpServletRequest) 
            FacesContext.getCurrentInstance().getExternalContext().getRequest();
        return (String) request.getSession().getAttribute("userId");
    }
    
    // Getters and setters
    public List<ActivityLog> getRealtimeActivities() { return realtimeActivities; }
    public List<SecurityAlert> getSecurityAlerts() { return securityAlerts; }
    public SecurityAlert getSelectedAlert() { return selectedAlert; }
    public LineChartModel getLoginTrendsModel() { return loginTrendsModel; }
    public PieChartModel getModuleUsageModel() { return moduleUsageModel; }
}
```

---

### 7. Usage Examples and Integration

#### How to Use These Filters:

1. **Automatic Activity Tracking**: 
   - Filters automatically capture all HTTP requests
   - No additional code needed in JSF pages
   - Works transparently with existing applications

2. **Document Access Tracking**:
   - Automatically tracks file views, downloads, uploads
   - Measures view duration for analytics
   - Integrates with PrimeFaces file components

3. **Module Usage Tracking**:
   - Phase listener tracks navigation between modules
   - Measures time spent in each module
   - Works with JSF navigation system

4. **Security Monitoring**:
   - Automatic detection of suspicious activities
   - Real-time alerts displayed in dashboard
   - Integration with existing security systems

#### Performance Considerations:

1. **Asynchronous Logging**: All activity logging is done asynchronously
2. **Static Resource Filtering**: Static resources are excluded from tracking
3. **Lightweight Operations**: Minimal performance impact on user experience
4. **Batch Processing**: Activities are batched and processed every 30 seconds

#### PDPA Compliance:

1. **IP Masking**: IP addresses are automatically masked
2. **User ID Anonymization**: User IDs can be hashed/anonymized
3. **Data Retention**: Automatic archival and deletion after specified periods
4. **Audit Trail**: Complete audit trail for compliance reporting

This implementation provides comprehensive activity tracking for JSF/PrimeFaces applications while maintaining performance and compliance requirements.
