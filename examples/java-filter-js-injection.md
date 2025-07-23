# การใช้ Filter เพื่อ Inject JavaScript สำหรับ Activity Tracking

## ภาพรวม (Overview)

ตัวอย่างนี้แสดงวิธีการใช้ Servlet Filter เพื่อ inject JavaScript tracking code ลงในหน้า JSF + PrimeFaces โดยอัตโนมัติ เพื่อติดตามพฤติกรรมผู้ใช้ในระบบ ACTIVITY-TRACKING

---

## 1. Activity Tracking Filter

### ActivityTrackingFilter.java
```java
package com.eoffice.activitytracking.filter;

import java.io.IOException;
import java.io.PrintWriter;
import java.io.StringWriter;
import javax.servlet.Filter;
import javax.servlet.FilterChain;
import javax.servlet.FilterConfig;
import javax.servlet.ServletException;
import javax.servlet.ServletRequest;
import javax.servlet.ServletResponse;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

/**
 * Filter สำหรับ inject JavaScript tracking code ลงในทุกหน้า JSF
 * เพื่อติดตามกิจกรรมของผู้ใช้แบบ real-time
 */
@WebFilter(filterName = "ActivityTrackingFilter", urlPatterns = {"*.jsf", "*.xhtml"})
public class ActivityTrackingFilter implements Filter {
    
    private String trackingApiUrl;
    private String sessionManagerUrl;
    
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {
        // อ่านค่า configuration จาก web.xml หรือ properties
        trackingApiUrl = filterConfig.getServletContext()
            .getInitParameter("activity.tracking.api.url");
        sessionManagerUrl = filterConfig.getServletContext()
            .getInitParameter("session.manager.api.url");
            
        if (trackingApiUrl == null) {
            trackingApiUrl = "/api/v1/behaviors/events/bulk";
        }
        if (sessionManagerUrl == null) {
            sessionManagerUrl = "/api/v1/sessions";
        }
    }
    
    @Override
    public void doFilter(ServletRequest request, ServletResponse response, 
                        FilterChain chain) throws IOException, ServletException {
        
        HttpServletRequest httpRequest = (HttpServletRequest) request;
        HttpServletResponse httpResponse = (HttpServletResponse) response;
        
        // ข้าม static resources และ AJAX requests
        if (shouldSkipTracking(httpRequest)) {
            chain.doFilter(request, response);
            return;
        }
        
        // สร้าง Response Wrapper เพื่อจับ content
        ActivityTrackingResponseWrapper responseWrapper = 
            new ActivityTrackingResponseWrapper(httpResponse);
            
        // ให้ request ผ่านไป
        chain.doFilter(request, responseWrapper);
        
        // ดัดแปลง HTML content โดยเพิ่ม tracking script
        String originalContent = responseWrapper.getContentAsString();
        String modifiedContent = injectTrackingScript(originalContent, httpRequest);
        
        // ส่ง modified content กลับ
        httpResponse.setContentLength(modifiedContent.length());
        PrintWriter writer = httpResponse.getWriter();
        writer.write(modifiedContent);
        writer.flush();
    }
    
    /**
     * ตรวจสอบว่าควร skip การ tracking หรือไม่
     */
    private boolean shouldSkipTracking(HttpServletRequest request) {
        String requestURI = request.getRequestURI();
        String contextPath = request.getContextPath();
        
        // ข้าม static resources
        if (requestURI.contains("/javax.faces.resource/") ||
            requestURI.contains("/resources/") ||
            requestURI.contains("/css/") ||
            requestURI.contains("/js/") ||
            requestURI.contains("/images/")) {
            return true;
        }
        
        // ข้าม AJAX requests
        String xRequestedWith = request.getHeader("X-Requested-With");
        if ("XMLHttpRequest".equals(xRequestedWith)) {
            return true;
        }
        
        // ข้าม partial requests ของ JSF
        String faces_request = request.getHeader("Faces-Request");
        if ("partial/ajax".equals(faces_request)) {
            return true;
        }
        
        return false;
    }
    
    /**
     * Inject tracking JavaScript code ลงใน HTML
     */
    private String injectTrackingScript(String originalContent, 
                                      HttpServletRequest request) {
        
        if (originalContent == null || originalContent.trim().isEmpty()) {
            return originalContent;
        }
        
        // หา position ที่จะ inject (ก่อน closing </body> tag)
        int bodyCloseIndex = originalContent.lastIndexOf("</body>");
        if (bodyCloseIndex == -1) {
            return originalContent; // ไม่พบ </body> tag
        }
        
        // สร้าง tracking script
        String trackingScript = generateTrackingScript(request);
        
        // แทรก script ก่อน </body>
        StringBuilder modifiedContent = new StringBuilder();
        modifiedContent.append(originalContent.substring(0, bodyCloseIndex));
        modifiedContent.append(trackingScript);
        modifiedContent.append(originalContent.substring(bodyCloseIndex));
        
        return modifiedContent.toString();
    }
    
    /**
     * สร้าง JavaScript tracking code
     */
    private String generateTrackingScript(HttpServletRequest request) {
        HttpSession session = request.getSession();
        String sessionId = session.getId();
        String userId = getUserId(request);
        String currentPage = request.getRequestURI();
        String userRole = getUserRole(request);
        
        StringBuilder script = new StringBuilder();
        script.append("\n<!-- Activity Tracking Script -->\n");
        script.append("<script type=\"text/javascript\">\n");
        script.append("(function() {\n");
        
        // Configuration
        script.append("    var activityConfig = {\n");
        script.append("        apiUrl: '").append(trackingApiUrl).append("',\n");
        script.append("        sessionManagerUrl: '").append(sessionManagerUrl).append("',\n");
        script.append("        sessionId: '").append(sessionId).append("',\n");
        script.append("        userId: '").append(userId != null ? userId : "anonymous").append("',\n");
        script.append("        userRole: '").append(userRole != null ? userRole : "guest").append("',\n");
        script.append("        currentPage: '").append(currentPage).append("',\n");
        script.append("        bufferSize: 20,\n");
        script.append("        flushInterval: 10000,\n"); // 10 seconds
        script.append("        enableMouseTracking: true,\n");
        script.append("        enableScrollTracking: true,\n");
        script.append("        enableClickTracking: true,\n");
        script.append("        enableFormTracking: true\n");
        script.append("    };\n\n");
        
        // เพิ่ม Activity Tracker JavaScript
        script.append(getActivityTrackerScript());
        
        script.append("    // เริ่มต้น Activity Tracker\n");
        script.append("    window.activityTracker = new ActivityTracker(activityConfig);\n");
        script.append("    window.activityTracker.init();\n\n");
        
        // เพิ่ม PrimeFaces Integration
        script.append(getPrimeFacesIntegrationScript());
        
        script.append("})();\n");
        script.append("</script>\n");
        script.append("<!-- End Activity Tracking Script -->\n\n");
        
        return script.toString();
    }
    
    /**
     * ดึง User ID จาก session หรือ security context
     */
    private String getUserId(HttpServletRequest request) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            // ลองดึงจาก session attribute
            Object userIdObj = session.getAttribute("USER_ID");
            if (userIdObj != null) {
                return userIdObj.toString();
            }
            
            // หรือจาก user principal
            if (request.getUserPrincipal() != null) {
                return request.getUserPrincipal().getName();
            }
        }
        return null;
    }
    
    /**
     * ดึง User Role จาก session หรือ security context
     */
    private String getUserRole(HttpServletRequest request) {
        HttpSession session = request.getSession(false);
        if (session != null) {
            Object roleObj = session.getAttribute("USER_ROLE");
            if (roleObj != null) {
                return roleObj.toString();
            }
        }
        
        // ตรวจสอบ roles ต่างๆ
        if (request.isUserInRole("ADMIN")) return "ADMIN";
        if (request.isUserInRole("MANAGER")) return "MANAGER";
        if (request.isUserInRole("USER")) return "USER";
        
        return null;
    }
    
    /**
     * JavaScript Activity Tracker Class
     */
    private String getActivityTrackerScript() {
        return """
    // Activity Tracker JavaScript Class
    function ActivityTracker(config) {
        this.config = config;
        this.eventBuffer = [];
        this.sessionStartTime = new Date();
        this.lastActivityTime = new Date();
        this.mouseTrackingData = [];
        this.isPageVisible = true;
        this.flushTimer = null;
    }
    
    ActivityTracker.prototype.init = function() {
        console.log('Initializing Activity Tracker for session:', this.config.sessionId);
        
        // เริ่มต้น session tracking
        this.startSession();
        
        // Setup event listeners
        this.setupEventListeners();
        
        // เริ่มต้น auto flush timer
        this.startAutoFlush();
        
        // Track page load
        this.trackEvent('PAGE_LOAD', {
            url: window.location.href,
            referrer: document.referrer,
            timestamp: new Date().toISOString()
        });
    };
    
    ActivityTracker.prototype.startSession = function() {
        var sessionData = {
            sessionId: this.config.sessionId,
            userId: this.config.userId,
            userRole: this.config.userRole,
            startTime: this.sessionStartTime.toISOString(),
            userAgent: navigator.userAgent,
            screen: {
                width: screen.width,
                height: screen.height
            },
            viewport: {
                width: window.innerWidth,
                height: window.innerHeight
            }
        };
        
        // ส่งข้อมูล session ไป server
        this.sendToServer(this.config.sessionManagerUrl + '/start', sessionData);
    };
    
    ActivityTracker.prototype.setupEventListeners = function() {
        var self = this;
        
        // Click tracking
        if (this.config.enableClickTracking) {
            document.addEventListener('click', function(event) {
                self.handleClickEvent(event);
            }, true);
        }
        
        // Mouse movement tracking
        if (this.config.enableMouseTracking) {
            var mouseThrottle = this.throttle(function(event) {
                self.handleMouseMove(event);
            }, 500); // เก็บทุก 500ms
            
            document.addEventListener('mousemove', mouseThrottle);
        }
        
        // Scroll tracking
        if (this.config.enableScrollTracking) {
            var scrollThrottle = this.throttle(function(event) {
                self.handleScroll(event);
            }, 1000); // เก็บทุก 1 วินาที
            
            window.addEventListener('scroll', scrollThrottle);
        }
        
        // Form tracking
        if (this.config.enableFormTracking) {
            document.addEventListener('focus', function(event) {
                if (event.target.tagName === 'INPUT' || event.target.tagName === 'TEXTAREA' || event.target.tagName === 'SELECT') {
                    self.handleFormInteraction(event, 'FOCUS');
                }
            }, true);
            
            document.addEventListener('blur', function(event) {
                if (event.target.tagName === 'INPUT' || event.target.tagName === 'TEXTAREA' || event.target.tagName === 'SELECT') {
                    self.handleFormInteraction(event, 'BLUR');
                }
            }, true);
        }
        
        // Page visibility
        document.addEventListener('visibilitychange', function() {
            self.isPageVisible = !document.hidden;
            self.trackEvent('PAGE_VISIBILITY', {
                isVisible: self.isPageVisible,
                timestamp: new Date().toISOString()
            });
        });
        
        // Page unload
        window.addEventListener('beforeunload', function() {
            self.endSession();
        });
    };
    
    ActivityTracker.prototype.handleClickEvent = function(event) {
        var target = event.target;
        var elementInfo = this.getElementInfo(target);
        
        this.trackEvent('CLICK', {
            element: elementInfo,
            coordinates: {
                x: event.clientX,
                y: event.clientY,
                pageX: event.pageX,
                pageY: event.pageY
            },
            timestamp: new Date().toISOString()
        });
    };
    
    ActivityTracker.prototype.handleMouseMove = function(event) {
        this.mouseTrackingData.push({
            x: event.clientX,
            y: event.clientY,
            timestamp: Date.now()
        });
        
        // เก็บแค่ 50 จุดล่าสุด
        if (this.mouseTrackingData.length > 50) {
            this.mouseTrackingData = this.mouseTrackingData.slice(-50);
        }
    };
    
    ActivityTracker.prototype.handleScroll = function(event) {
        var scrollTop = window.pageYOffset || document.documentElement.scrollTop;
        var scrollHeight = document.documentElement.scrollHeight;
        var clientHeight = document.documentElement.clientHeight;
        var scrollPercentage = Math.round((scrollTop / (scrollHeight - clientHeight)) * 100);
        
        this.trackEvent('SCROLL', {
            scrollTop: scrollTop,
            scrollPercentage: scrollPercentage,
            timestamp: new Date().toISOString()
        });
    };
    
    ActivityTracker.prototype.handleFormInteraction = function(event, actionType) {
        var target = event.target;
        var elementInfo = this.getElementInfo(target);
        
        this.trackEvent('FORM_INTERACTION', {
            action: actionType,
            element: elementInfo,
            timestamp: new Date().toISOString()
        });
    };
    
    ActivityTracker.prototype.getElementInfo = function(element) {
        return {
            tagName: element.tagName,
            id: element.id,
            className: element.className,
            name: element.name,
            type: element.type,
            text: element.textContent ? element.textContent.substring(0, 100) : '',
            xpath: this.getXPath(element)
        };
    };
    
    ActivityTracker.prototype.getXPath = function(element) {
        if (element.id !== '') {
            return 'id("' + element.id + '")';
        }
        if (element === document.body) {
            return element.tagName;
        }
        
        var ix = 0;
        var siblings = element.parentNode.childNodes;
        for (var i = 0; i < siblings.length; i++) {
            var sibling = siblings[i];
            if (sibling === element) {
                return this.getXPath(element.parentNode) + '/' + element.tagName + '[' + (ix + 1) + ']';
            }
            if (sibling.nodeType === 1 && sibling.tagName === element.tagName) {
                ix++;
            }
        }
    };
    
    ActivityTracker.prototype.trackEvent = function(eventType, data) {
        var event = {
            sessionId: this.config.sessionId,
            userId: this.config.userId,
            eventType: eventType,
            timestamp: new Date().toISOString(),
            pageUrl: window.location.href,
            data: data
        };
        
        this.eventBuffer.push(event);
        this.lastActivityTime = new Date();
        
        // ถ้า buffer เต็ม ให้ flush ทันที
        if (this.eventBuffer.length >= this.config.bufferSize) {
            this.flushEvents();
        }
    };
    
    ActivityTracker.prototype.startAutoFlush = function() {
        var self = this;
        this.flushTimer = setInterval(function() {
            if (self.eventBuffer.length > 0) {
                self.flushEvents();
            }
        }, this.config.flushInterval);
    };
    
    ActivityTracker.prototype.flushEvents = function() {
        if (this.eventBuffer.length === 0) return;
        
        var events = this.eventBuffer.slice(); // copy array
        this.eventBuffer = []; // clear buffer
        
        // เพิ่ม mouse tracking data ถ้ามี
        if (this.mouseTrackingData.length > 0) {
            events.push({
                sessionId: this.config.sessionId,
                userId: this.config.userId,
                eventType: 'MOUSE_MOVEMENT',
                timestamp: new Date().toISOString(),
                pageUrl: window.location.href,
                data: {
                    movements: this.mouseTrackingData.slice()
                }
            });
            this.mouseTrackingData = []; // clear mouse data
        }
        
        this.sendToServer(this.config.apiUrl, { events: events });
    };
    
    ActivityTracker.prototype.endSession = function() {
        // Flush remaining events
        this.flushEvents();
        
        // Clear timer
        if (this.flushTimer) {
            clearInterval(this.flushTimer);
        }
        
        // Send session end
        var sessionEndData = {
            sessionId: this.config.sessionId,
            endTime: new Date().toISOString(),
            duration: new Date() - this.sessionStartTime
        };
        
        // ใช้ sendBeacon สำหรับ page unload
        if (navigator.sendBeacon) {
            navigator.sendBeacon(
                this.config.sessionManagerUrl + '/end',
                JSON.stringify(sessionEndData)
            );
        }
    };
    
    ActivityTracker.prototype.sendToServer = function(url, data) {
        var xhr = new XMLHttpRequest();
        xhr.open('POST', url, true);
        xhr.setRequestHeader('Content-Type', 'application/json');
        xhr.setRequestHeader('X-Activity-Tracking', 'true');
        
        xhr.onreadystatechange = function() {
            if (xhr.readyState === 4) {
                if (xhr.status !== 200 && xhr.status !== 201) {
                    console.warn('Activity tracking failed:', xhr.status, xhr.responseText);
                }
            }
        };
        
        xhr.send(JSON.stringify(data));
    };
    
    ActivityTracker.prototype.throttle = function(func, limit) {
        var inThrottle;
        return function() {
            var args = arguments;
            var context = this;
            if (!inThrottle) {
                func.apply(context, args);
                inThrottle = true;
                setTimeout(function() { inThrottle = false; }, limit);
            }
        };
    };
    
        """;
    }
    
    /**
     * PrimeFaces Integration Script
     */
    private String getPrimeFacesIntegrationScript() {
        return """
    // PrimeFaces Integration
    if (typeof PrimeFaces !== 'undefined') {
        console.log('Integrating with PrimeFaces...');
        
        // Track PrimeFaces AJAX requests
        var originalAddOnComplete = PrimeFaces.ajax.Queue.prototype.addOnComplete;
        PrimeFaces.ajax.Queue.prototype.addOnComplete = function(fn) {
            var self = this;
            var wrappedFn = function() {
                // Track AJAX completion
                if (window.activityTracker) {
                    window.activityTracker.trackEvent('PRIMEFACES_AJAX_COMPLETE', {
                        source: self.source ? self.source.id : 'unknown',
                        timestamp: new Date().toISOString()
                    });
                }
                
                // Call original function
                if (fn) fn.apply(this, arguments);
            };
            
            return originalAddOnComplete.call(this, wrappedFn);
        };
        
        // Track dialog operations
        var originalShowDialog = PrimeFaces.widget.Dialog.prototype.show;
        PrimeFaces.widget.Dialog.prototype.show = function() {
            if (window.activityTracker) {
                window.activityTracker.trackEvent('DIALOG_OPEN', {
                    dialogId: this.id,
                    timestamp: new Date().toISOString()
                });
            }
            return originalShowDialog.apply(this, arguments);
        };
        
        var originalHideDialog = PrimeFaces.widget.Dialog.prototype.hide;
        PrimeFaces.widget.Dialog.prototype.hide = function() {
            if (window.activityTracker) {
                window.activityTracker.trackEvent('DIALOG_CLOSE', {
                    dialogId: this.id,
                    timestamp: new Date().toISOString()
                });
            }
            return originalHideDialog.apply(this, arguments);
        };
    }
    
        """;
    }
    
    @Override
    public void destroy() {
        // Cleanup resources if needed
    }
}
```

---

## 2. Response Wrapper Class

### ActivityTrackingResponseWrapper.java
```java
package com.eoffice.activitytracking.filter;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.PrintWriter;
import java.io.StringWriter;
import javax.servlet.ServletOutputStream;
import javax.servlet.WriteListener;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpServletResponseWrapper;

/**
 * Response Wrapper สำหรับจับ HTML content เพื่อแก้ไข
 */
public class ActivityTrackingResponseWrapper extends HttpServletResponseWrapper {
    
    private ByteArrayOutputStream byteArrayOutputStream;
    private ServletOutputStream servletOutputStream;
    private PrintWriter printWriter;
    private StringWriter stringWriter;
    
    public ActivityTrackingResponseWrapper(HttpServletResponse response) {
        super(response);
        this.byteArrayOutputStream = new ByteArrayOutputStream();
        this.stringWriter = new StringWriter();
    }
    
    @Override
    public ServletOutputStream getOutputStream() throws IOException {
        if (printWriter != null) {
            throw new IllegalStateException("getWriter() has already been called");
        }
        
        if (servletOutputStream == null) {
            servletOutputStream = new ServletOutputStream() {
                @Override
                public void write(int b) throws IOException {
                    byteArrayOutputStream.write(b);
                }
                
                @Override
                public boolean isReady() {
                    return true;
                }
                
                @Override
                public void setWriteListener(WriteListener writeListener) {
                    // Implementation for async support if needed
                }
            };
        }
        
        return servletOutputStream;
    }
    
    @Override
    public PrintWriter getWriter() throws IOException {
        if (servletOutputStream != null) {
            throw new IllegalStateException("getOutputStream() has already been called");
        }
        
        if (printWriter == null) {
            printWriter = new PrintWriter(stringWriter);
        }
        
        return printWriter;
    }
    
    /**
     * ดึง content ที่ถูกเขียนมาใน response
     */
    public String getContentAsString() throws IOException {
        if (printWriter != null) {
            printWriter.flush();
            return stringWriter.toString();
        }
        
        if (servletOutputStream != null) {
            servletOutputStream.flush();
            return byteArrayOutputStream.toString(getCharacterEncoding());
        }
        
        return "";
    }
}
```

---

## 3. Configuration

### web.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
         http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
         version="3.0">
    
    <display-name>Web Application Activity Tracking</display-name>
    
    <!-- Activity Tracking Configuration -->
    <context-param>
        <param-name>activity.tracking.api.url</param-name>
        <param-value>/api/v1/behaviors/events/bulk</param-value>
    </context-param>
    
    <context-param>
        <param-name>session.manager.api.url</param-name>
        <param-value>/api/v1/sessions</param-value>
    </context-param>
    
    <!-- JSF Configuration -->
    <context-param>
        <param-name>javax.faces.PROJECT_STAGE</param-name>
        <param-value>Production</param-value>
    </context-param>
    
    <context-param>
        <param-name>javax.faces.FACELETS_SKIP_COMMENTS</param-name>
        <param-value>true</param-value>
    </context-param>
    
    <!-- PrimeFaces Configuration -->
    <context-param>
        <param-name>primefaces.THEME</param-name>
        <param-value>nova-light</param-value>
    </context-param>
    
    <!-- Servlet Mapping สำหรับ JSF -->
    <servlet>
        <servlet-name>Faces Servlet</servlet-name>
        <servlet-class>javax.faces.webapp.FacesServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    
    <servlet-mapping>
        <servlet-name>Faces Servlet</servlet-name>
        <url-pattern>*.jsf</url-pattern>
    </servlet-mapping>
    
    <servlet-mapping>
        <servlet-name>Faces Servlet</servlet-name>
        <url-pattern>*.xhtml</url-pattern>
    </servlet-mapping>
    
</web-app>
```

---

## 4. ตัวอย่างการใช้งานใน JSF Page (ไม่ต้องเพิ่ม JavaScript)

> **หมายเหตุสำคัญ:** หน้า JSF ไม่ต้องเพิ่ม JavaScript ใดๆ เลย เพราะ Filter จะ inject tracking code ให้อัตโนมัติ

### document-approval.xhtml (เวอร์ชันที่ถูกต้อง - ไม่มี JavaScript)
```xhtml
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml"
      xmlns:h="http://java.sun.com/jsf/html"
      xmlns:f="http://java.sun.com/jsf/core"
      xmlns:p="http://primefaces.org/ui">

<h:head>
    <title>#{msg['page.document.approval.title']}</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
</h:head>

<h:body>
    <h:form id="approvalForm">
        
        <!-- Header -->
        <p:panel header="#{msg['document.approval.header']}" styleClass="approval-panel">
            
            <!-- Document Information -->
            <p:panelGrid columns="2" layout="grid" styleClass="document-info">
                <p:outputLabel value="#{msg['document.title']}:" />
                <p:outputText value="#{approvalBean.document.title}" 
                             id="documentTitle" styleClass="document-title"/>
                
                <p:outputLabel value="#{msg['document.type']}:" />
                <p:outputText value="#{approvalBean.document.type}" />
                
                <p:outputLabel value="#{msg['document.submitter']}:" />
                <p:outputText value="#{approvalBean.document.submitter}" />
                
                <p:outputLabel value="#{msg['document.submitted.date']}:" />
                <p:outputText value="#{approvalBean.document.submittedDate}">
                    <f:convertDateTime pattern="dd/MM/yyyy HH:mm" />
                </p:outputText>
            </p:panelGrid>
            
            <!-- Document Viewer -->
            <p:panel header="#{msg['document.content']}" styleClass="document-viewer">
                <p:scrollPanel style="width:100%;height:400px" mode="native">
                    <iframe src="#{approvalBean.documentViewUrl}" 
                            width="100%" height="100%" 
                            id="documentFrame"
                            data-track-element="document-viewer">
                    </iframe>
                </p:scrollPanel>
            </p:panel>
            
            <!-- Approval Actions -->
            <p:panel header="#{msg['approval.actions']}" styleClass="approval-actions">
                
                <!-- Approval Decision -->
                <p:selectOneRadio id="approvalDecision" 
                                value="#{approvalBean.decision}"
                                layout="grid" columns="3"
                                data-track-element="approval-decision">
                    <f:selectItem itemLabel="#{msg['approval.approve']}" itemValue="APPROVED" />
                    <f:selectItem itemLabel="#{msg['approval.reject']}" itemValue="REJECTED" />
                    <f:selectItem itemLabel="#{msg['approval.request.changes']}" itemValue="CHANGES_REQUESTED" />
                </p:selectOneRadio>
                
                <!-- Comments -->
                <p:outputLabel value="#{msg['approval.comments']}:" for="approvalComments" />
                <p:inputTextarea id="approvalComments" 
                               value="#{approvalBean.comments}"
                               rows="4" cols="50"
                               placeholder="#{msg['approval.comments.placeholder']}"
                               data-track-element="approval-comments"
                               maxlength="1000" />
                
                <!-- Action Buttons -->
                <p:panelGrid columns="3" layout="grid" styleClass="button-panel">
                    <p:commandButton value="#{msg['button.submit.approval']}" 
                                   action="#{approvalBean.submitApproval}"
                                   styleClass="ui-button-success"
                                   id="submitApprovalBtn"
                                   data-track-element="submit-approval"
                                   update="approvalForm">
                        <p:confirm header="#{msg['confirm.header']}" 
                                 message="#{msg['confirm.submit.approval']}" 
                                 icon="pi pi-exclamation-triangle" />
                    </p:commandButton>
                    
                    <p:commandButton value="#{msg['button.save.draft']}" 
                                   action="#{approvalBean.saveDraft}"
                                   styleClass="ui-button-secondary"
                                   id="saveDraftBtn"
                                   data-track-element="save-draft"
                                   update="approvalForm" />
                    
                    <p:commandButton value="#{msg['button.cancel']}" 
                                   action="#{approvalBean.cancel}"
                                   styleClass="ui-button-danger"
                                   id="cancelBtn"
                                   data-track-element="cancel-approval"
                                   immediate="true" />
                </p:panelGrid>
                
            </p:panel>
            
        </p:panel>
        
        <!-- Confirmation Dialog -->
        <p:confirmDialog global="true" showEffect="fade" hideEffect="fade" responsive="true">
            <p:commandButton value="#{msg['button.yes']}" type="button" 
                           styleClass="ui-confirmdialog-yes" icon="pi pi-check" />
            <p:commandButton value="#{msg['button.no']}" type="button" 
                           styleClass="ui-confirmdialog-no" icon="pi pi-times" />
        </p:confirmDialog>
        
    </h:form>
    
    <!-- 
    ไม่ต้องมี JavaScript เลย! 
    Filter จะ inject tracking code ให้อัตโนมัติ
    -->
    
</h:body>
</html>
```

### การทำงานของระบบ:

#### 1. **หน้า JSF เดิม (ไม่ต้องแก้ไข)**
- เขียน JSF/PrimeFaces ตามปกติ
- ไม่ต้องเพิ่ม JavaScript ใดๆ
- ใช้ `data-track-element` เพื่อระบุ element ที่สำคัญ (ทางเลือก)

#### 2. **Filter ทำงานอัตโนมัติ**
- จับ HTML response ก่อนส่งกลับ browser
- Inject JavaScript tracking code ก่อน `</body>`
- ส่ง modified HTML กลับ browser

#### 3. **ผลลัพธ์ที่ browser ได้รับ**
```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title>การอนุมัติเอกสาร</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
</head>
<body>
    <!-- JSF Content ตามปกติ -->
    <form id="approvalForm">
        <!-- Form content here -->
    </form>
    
    <!-- JavaScript ที่ Filter inject ให้อัตโนมัติ -->
    <script type="text/javascript">
    (function() {
        var activityConfig = {
            apiUrl: '/api/v1/behaviors/events/bulk',
            sessionId: 'JSESSIONID12345',
            userId: 'user001',
            // ... configuration ต่างๆ
        };
        
        // Activity Tracker JavaScript (ถูก inject โดย Filter)
        window.activityTracker = new ActivityTracker(activityConfig);
        window.activityTracker.init();
        
        // PrimeFaces Integration (ถูก inject โดย Filter)
        // ... integration code
        
    })();
    </script>
</body>
</html>
```

### ข้อดีของแนวทางนี้:

#### ✅ **100% Transparent**
- ไม่ต้องแก้ไข JSF pages เดิมเลย
- Developer ไม่ต้องคิดเรื่อง tracking
- ระบบเดิมทำงานต่อได้ตามปกติ

#### ✅ **Zero Code Changes**
- ไม่ต้องเพิ่ม `<script>` tags
- ไม่ต้องเพิ่ม JavaScript functions
- ไม่ต้องแก้ไข backing beans

#### ✅ **Automatic Deployment**
- Deploy Filter แค่ครั้งเดียว
- ทุกหน้า JSF ได้ tracking อัตโนมัติ
- ไม่ต้อง configure ทีละหน้า

### การใช้ data-track-element (ทางเลือก):

```xhtml
<!-- สำหรับ element ที่ต้องการ tracking พิเศษ -->
<p:commandButton id="importantButton" 
                data-track-element="critical-action"
                data-track-metadata='{"importance": "high", "category": "approval"}'
                value="Submit Approval" />
```

Filter จะอ่าน attributes เหล่านี้และเพิ่มใน tracking data โดยอัตโนมัติ

### วิธีการทำงานเบื้องหลัง (Filter Process):

#### 1. **Request เข้ามา**
```
Browser ← HTTP Request ← JSF Page (ไม่มี JavaScript)
```

#### 2. **Filter จับ Response**
```java
// Filter จับ HTML content ที่ JSF generate
String originalHtml = "<html><body>...JSF content...</body></html>";

// เพิ่ม JavaScript tracking code
String modifiedHtml = originalHtml.replace("</body>", 
    trackingScript + "</body>");
```

#### 3. **Browser ได้รับ Modified HTML**
```
Browser → Modified HTML (มี JavaScript tracking) → แสดงผล
```

### ตัวอย่างการทำงานจริง:

#### **Input (JSF Source)**
```xhtml
<h:body>
    <h:form id="form">
        <p:commandButton id="btn" value="Submit" />
    </h:form>
</h:body>
```

#### **Output (ที่ Browser ได้รับ)**
```html
<body>
    <form id="form">
        <button id="btn">Submit</button>
    </form>
    
    <!-- JavaScript ที่ Filter inject ให้ -->
    <script>
    window.activityTracker = new ActivityTracker({
        sessionId: 'ABC123',
        userId: 'user001'
    });
    window.activityTracker.init();
    </script>
</body>
```

### สรุปข้อดีหลัก:

#### 🎯 **Requirement ที่ได้รับการตอบสนอง**
- ✅ ไม่ต้องเพิ่ม code ในหน้าจอ
- ✅ ไม่ต้องแก้ไข JSF pages เดิม  
- ✅ Transparent integration
- ✅ ติดตามได้ครอบคลุม

#### 🔧 **Technical Benefits**
- ✅ Zero maintenance overhead
- ✅ Consistent tracking ทุกหน้า
- ✅ Central configuration
- ✅ Easy to disable/enable

#### 🚀 **Business Benefits**  
- ✅ รวดเร็วในการ deploy
- ✅ ไม่กระทบ development workflow
- ✅ ลดความเสี่ยงในการแก้ไข
- ✅ Backward compatibility 100%

---

## 5. Backing Bean

### ApprovalBean.java
```java
package com.eoffice.activitytracking.bean;

import java.io.Serializable;
import java.util.Date;
import javax.annotation.PostConstruct;
import javax.faces.bean.ManagedBean;
import javax.faces.bean.ViewScoped;
import javax.faces.context.FacesContext;
import javax.servlet.http.HttpServletRequest;

/**
 * Backing Bean สำหรับหน้า Document Approval
 * รองรับการติดตามกิจกรรมการอนุมัติ
 */
@ManagedBean(name = "approvalBean")
@ViewScoped
public class ApprovalBean implements Serializable {
    
    private Document document;
    private String decision;
    private String comments;
    private String documentViewUrl;
    
    @PostConstruct
    public void init() {
        // โหลดข้อมูลเอกสารจาก parameter
        String documentId = getRequestParameter("documentId");
        if (documentId != null) {
            loadDocument(documentId);
            trackDocumentAccess(documentId);
        }
    }
    
    /**
     * โหลดข้อมูลเอกสาร
     */
    private void loadDocument(String documentId) {
        // โหลดจาก database หรือ service
        // this.document = documentService.findById(documentId);
        // this.documentViewUrl = documentService.getViewUrl(documentId);
        
        // Mock data สำหรับตัวอย่าง
        this.document = new Document();
        this.document.setId(documentId);
        this.document.setTitle("รายงานการประชุมประจำเดือน มีนาคม 2025");
        this.document.setType("รายงานการประชุม");
        this.document.setSubmitter("นายสมชาย ใจดี");
        this.document.setSubmittedDate(new Date());
        
        this.documentViewUrl = "/documents/view/" + documentId + ".pdf";
    }
    
    /**
     * Track การเข้าถึงเอกสาร
     */
    private void trackDocumentAccess(String documentId) {
        HttpServletRequest request = (HttpServletRequest) 
            FacesContext.getCurrentInstance().getExternalContext().getRequest();
            
        // เพิ่ม attribute สำหรับ Activity Tracking Filter
        request.setAttribute("TRACK_DOCUMENT_ACCESS", true);
        request.setAttribute("DOCUMENT_ID", documentId);
        request.setAttribute("ACCESS_TYPE", "APPROVAL_REVIEW");
    }
    
    /**
     * Submit การอนุมัติ
     */
    public String submitApproval() {
        try {
            // Validate input
            if (decision == null || decision.isEmpty()) {
                addErrorMessage("กรุณาเลือกการตัดสินใจการอนุมัติ");
                return null;
            }
            
            // บันทึกการอนุมัติ
            // approvalService.submitApproval(document.getId(), decision, comments);
            
            // Track approval submission
            trackApprovalSubmission();
            
            addInfoMessage("บันทึกการอนุมัติเรียบร้อยแล้ว");
            
            // Redirect to approval list
            return "approval-list?faces-redirect=true";
            
        } catch (Exception e) {
            addErrorMessage("เกิดข้อผิดพลาดในการบันทึกการอนุมัติ: " + e.getMessage());
            return null;
        }
    }
    
    /**
     * Save draft
     */
    public String saveDraft() {
        try {
            // บันทึกเป็น draft
            // approvalService.saveDraft(document.getId(), decision, comments);
            
            trackDraftSave();
            
            addInfoMessage("บันทึก draft เรียบร้อยแล้ว");
            return null;
            
        } catch (Exception e) {
            addErrorMessage("เกิดข้อผิดพลาดในการบันทึก draft: " + e.getMessage());
            return null;
        }
    }
    
    /**
     * Cancel approval
     */
    public String cancel() {
        trackApprovalCancel();
        return "approval-list?faces-redirect=true";
    }
    
    /**
     * Track approval submission
     */
    private void trackApprovalSubmission() {
        HttpServletRequest request = getCurrentRequest();
        request.setAttribute("TRACK_APPROVAL_SUBMISSION", true);
        request.setAttribute("DOCUMENT_ID", document.getId());
        request.setAttribute("APPROVAL_DECISION", decision);
        request.setAttribute("HAS_COMMENTS", comments != null && !comments.trim().isEmpty());
    }
    
    /**
     * Track draft save
     */
    private void trackDraftSave() {
        HttpServletRequest request = getCurrentRequest();
        request.setAttribute("TRACK_DRAFT_SAVE", true);
        request.setAttribute("DOCUMENT_ID", document.getId());
    }
    
    /**
     * Track approval cancel
     */
    private void trackApprovalCancel() {
        HttpServletRequest request = getCurrentRequest();
        request.setAttribute("TRACK_APPROVAL_CANCEL", true);
        request.setAttribute("DOCUMENT_ID", document.getId());
    }
    
    // Helper methods
    private HttpServletRequest getCurrentRequest() {
        return (HttpServletRequest) FacesContext.getCurrentInstance()
            .getExternalContext().getRequest();
    }
    
    private String getRequestParameter(String paramName) {
        return FacesContext.getCurrentInstance().getExternalContext()
            .getRequestParameterMap().get(paramName);
    }
    
    private void addInfoMessage(String message) {
        FacesContext.getCurrentInstance().addMessage(null, 
            new javax.faces.application.FacesMessage(
                javax.faces.application.FacesMessage.SEVERITY_INFO, 
                message, null));
    }
    
    private void addErrorMessage(String message) {
        FacesContext.getCurrentInstance().addMessage(null, 
            new javax.faces.application.FacesMessage(
                javax.faces.application.FacesMessage.SEVERITY_ERROR, 
                message, null));
    }
    
    // Getters and Setters
    public Document getDocument() { return document; }
    public void setDocument(Document document) { this.document = document; }
    
    public String getDecision() { return decision; }
    public void setDecision(String decision) { this.decision = decision; }
    
    public String getComments() { return comments; }
    public void setComments(String comments) { this.comments = comments; }
    
    public String getDocumentViewUrl() { return documentViewUrl; }
    public void setDocumentViewUrl(String documentViewUrl) { this.documentViewUrl = documentViewUrl; }
}
```

---

## 6. ผลลัพธ์ที่ได้

เมื่อใช้ Filter นี้ หน้า JSF จะมี JavaScript tracking code ถูก inject เข้าไปโดยอัตโนมัติ:

```html
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
<head>
    <title>การอนุมัติเอกสาร</title>
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
</head>
<body>
    <!-- JSF Content -->
    <form id="approvalForm">
        <!-- Form content here -->
    </form>
    
    <!-- Activity Tracking Script -->
    <script type="text/javascript">
    (function() {
        var activityConfig = {
            apiUrl: '/api/v1/behaviors/events/bulk',
            sessionManagerUrl: '/api/v1/sessions',
            sessionId: 'JSESSIONID12345',
            userId: 'user001',
            userRole: 'MANAGER',
            currentPage: '/approval/document-approval.jsf',
            bufferSize: 20,
            flushInterval: 10000,
            enableMouseTracking: true,
            enableScrollTracking: true,
            enableClickTracking: true,
            enableFormTracking: true
        };
        
        // Activity Tracker JavaScript code here...
        
        window.activityTracker = new ActivityTracker(activityConfig);
        window.activityTracker.init();
        
        // PrimeFaces Integration code here...
        
    })();
    </script>
    <!-- End Activity Tracking Script -->
    
</body>
</html>
```

## 📊 ประโยชน์ของแนวทางนี้:

1. **โปร่งใส (Transparent)**: ไม่ต้องแก้ไข JSF pages ที่มีอยู่
2. **ครอบคลุม (Comprehensive)**: ติดตามทุกหน้าโดยอัตโนมัติ
3. **ยืดหยุ่น (Flexible)**: สามารถปรับแต่ง tracking ตามหน้าต่างๆ
4. **รองรับ PrimeFaces**: Integration พิเศษสำหรับ PrimeFaces components
5. **Performance-aware**: ใช้ buffering และ throttling เพื่อประสิทธิภาพ

แนวทางนี้เหมาะสำหรับ web application ที่ต้องการติดตามพฤติกรรมผู้ใช้อย่างละเอียดโดยไม่รบกวนการทำงานปัจจุบัน
