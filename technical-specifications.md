# Technical Specifications - ACTIVITY-TRACKING System

## Document Information
**Project:** ACTIVITY-TRACKING System  
**Version:** 1.0  
**Date:** December 2024  
**Technology Stack:** Java JDK 1.8, JSF 2.x, PrimeFaces 8.x+, ELK Stack  
**Production Environment:** Multi-System Architecture (EDCS, LEAV, OFC-APP)

---

## 1. API Specifications

### 1.1 REST API Endpoints for Log Analytics

#### 1.1.1 Authentication & Session APIs
```java
// Base URL: /api/v1/analytics

// Get current active sessions from log analysis
GET /sessions/active
Response: {
  "activeUsers": 145,
  "totalSessions": 289,
  "averageSessionDuration": "25.3 minutes",
  "sessions": [
    {
      "sessionId": "dyw6WsNWH7V1qb9MWYcIHT2W1Xch9S4rOtKhxXSdcfSf8RMlIfH7!1170532732!1753325728341",
      "username": "sirat",
      "system": "edcs",
      "startTime": "2024-12-25T09:55:26.084Z",
      "lastActivity": "2024-12-25T10:15:42.123Z",
      "ipAddress": "172.17.3.***", // Masked for privacy
      "currentPage": "/edcs/pages/edcs/app/process/home/home.xhtml"
    }
  ]
}

// Get user authentication history from logs
GET /auth/history?userId={userId}&days={days}
Response: {
  "userId": "sirat",
  "period": "7 days",
  "loginAttempts": [
    {
      "timestamp": "2024-12-25T09:55:26.084Z",
      "system": "ofc-app", 
      "status": "success",
      "ipAddress": "172.17.3.***",
      "sessionId": "dyw6WsNWH7V1qb9MWYcIHT2W1Xch9S4rOtKhxXSdcfSf8RMlIfH7!1170532732!1753325683542"
    }
  ],
  "totalLogins": 23,
  "failedAttempts": 1,
  "averageSessionTime": "28.5 minutes"
}

// Get real-time login activities
GET /auth/realtime
Response: {
  "recentLogins": [
    {
      "username": "admin_ofc",
      "system": "leav",
      "timestamp": "2024-12-25T16:15:20.908Z",
      "action": "login",
      "sessionId": "xgQxamG2aXrEwBJKjrvJgfYRHJ2N6G51e7QU-xcDs95-l6K_tK0y!1170532732!1753175720907"
    }
  ],
  "loginTrends": {
    "hourly": [12, 15, 18, 22, 25, 30, 28],
    "peakHour": "09:00-10:00",
    "totalToday": 156
  }
}
```

#### 1.1.2 Document & Module Access APIs
```java
// Get document access patterns from log analysis
GET /documents/access?system={system}&module={module}&period={period}
Response: {
  "system": "edcs",
  "module": "search-letter", 
  "period": "24 hours",
  "totalAccess": 1247,
  "uniqueUsers": 89,
  "topPages": [
    {
      "page": "/edcs/pages/edcs/app/process/home/home.xhtml",
      "accessCount": 245,
      "uniqueUsers": 67,
      "averageDuration": "3.2 minutes"
    },
    {
      "page": "/edcs/pages/edcs/app/process/entryLetterIn/entryLetterIn.xhtml", 
      "accessCount": 189,
      "uniqueUsers": 45,
      "averageDuration": "5.8 minutes"
    }
  ],
  "accessTrends": {
    "hourly": [45, 67, 89, 123, 156, 189, 167, 145],
    "peakTime": "14:00-15:00"
  }
}

// Get module usage statistics across all systems
GET /modules/usage?period={period}
Response: {
  "period": "7 days",
  "systemUsage": {
    "edcs": {
      "totalAccess": 8945,
      "uniqueUsers": 234,
      "modules": {
        "process/home": 2345,
        "process/search-letter": 1876,
        "process/entry-letter-in": 1234,
        "process/entry-letter-out": 987
      }
    },
    "leav": {
      "totalAccess": 3456,
      "uniqueUsers": 189,
      "modules": {
        "leav-cancel-pending-all": 1234,
        "leav-request": 987,
        "leav-approve": 765
      }
    },
    "ofc-app": {
      "totalAccess": 5678,
      "uniqueUsers": 345,
      "modules": {
        "welcome": 2345,
        "profile": 1234,
        "settings": 789
      }
    }
  },
  "crossSystemNavigation": [
    {
      "pattern": "ofc-app → edcs → leav",
      "count": 89,
      "averageTime": "12.3 minutes"
    }
  ]
}

// Get real-time module activity
GET /modules/realtime
Response: {
  "currentActivity": [
    {
      "timestamp": "2024-12-25T16:15:20.908Z",
      "username": "admin_ofc",
      "system": "leav",
      "module": "leav-cancel-pending-all",
      "action": "page_access",
      "sessionId": "xgQxamG2aXrEwBJKjrvJgfYRHJ2N6G51e7QU-xcDs95-l6K_tK0y!1170532732!1753175720907"
    }
  ],
  "systemLoad": {
    "edcs": { "activeUsers": 67, "requests": 234 },
    "leav": { "activeUsers": 23, "requests": 89 },
    "ofc-app": { "activeUsers": 156, "requests": 445 }
  }
}
```

#### 1.1.3 System Performance & Health APIs
```java
// Get system performance metrics from log analysis
GET /system/performance?system={system}&metric={metric}
Response: {
  "system": "edcs",
  "timestamp": "2024-12-25T16:30:00.000Z",
  "metrics": {
    "responseTime": {
      "average": "1.2 seconds",
      "p95": "2.8 seconds",
      "p99": "4.5 seconds"
    },
    "requestVolume": {
      "current": 234,
      "peak": 456,
      "average": 189
    },
    "errorRate": {
      "rate": "0.02%",
      "totalErrors": 3,
      "totalRequests": 15678
    },
    "memoryUsage": {
      "heap": "4022.00 MiB",
      "used": "3424.79 MiB",
      "percentage": "85.1%"
    }
  },
  "trends": {
    "hourly": [189, 234, 267, 345, 456, 389, 234],
    "alertThreshold": 400,
    "status": "normal"
  }
}

// Get cross-system health dashboard
GET /system/health
Response: {
  "overall": {
    "status": "healthy",
    "uptime": "99.8%",
    "lastUpdate": "2024-12-25T16:30:15.000Z"
  },
  "systems": {
    "edcs": {
      "status": "healthy",
      "activeUsers": 67,
      "responseTime": "1.2s",
      "errorRate": "0.02%",
      "lastLogEntry": "2024-12-25T16:30:14.123Z"
    },
    "leav": {
      "status": "healthy", 
      "activeUsers": 23,
      "responseTime": "0.8s",
      "errorRate": "0.01%",
      "lastLogEntry": "2024-12-25T16:30:12.456Z"
    },
    "ofc-app": {
      "status": "healthy",
      "activeUsers": 156,
      "responseTime": "0.9s", 
      "errorRate": "0.03%",
      "lastLogEntry": "2024-12-25T16:30:13.789Z"
    }
  },
  "alerts": []
}
```

### 1.2 Elasticsearch Query Specifications

#### 1.2.1 Complex Log Search Queries
```json
// Production Session Tracking Query
{
  "query": {
    "bool": {
      "must": [
        {
          "range": {
            "@timestamp": {
              "gte": "now-1h",
              "lte": "now"
            }
          }
        },
        {
          "regexp": {
            "session_id": "[a-zA-Z0-9!_-]{50,70}"
          }
        }
      ],
      "should": [
        {
          "match": {
            "log_source": "edcs"
          }
        },
        {
          "match": {
            "log_source": "leav"
          }
        },
        {
          "match": {
            "log_source": "ofc-app"
          }
        }
      ]
    }
  },
  "aggs": {
    "sessions_by_user": {
      "terms": {
        "field": "username.keyword",
        "size": 100
      },
      "aggs": {
        "session_count": {
          "cardinality": {
            "field": "session_id.keyword"
          }
        },
        "systems_accessed": {
          "terms": {
            "field": "system.keyword"
          }
        }
      }
    },
    "activity_timeline": {
      "date_histogram": {
        "field": "@timestamp",
        "fixed_interval": "5m"
      },
      "aggs": {
        "unique_sessions": {
          "cardinality": {
            "field": "session_id.keyword"
          }
        }
      }
    }
  }
}

// Cross-System User Journey Analysis
{
  "query": {
    "bool": {
      "must": [
        {
          "exists": {
            "field": "session_id"
          }
        },
        {
          "exists": {
            "field": "system"
          }
        },
        {
          "range": {
            "@timestamp": {
              "gte": "now-24h"
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "user_journeys": {
      "terms": {
        "field": "session_id.keyword",
        "size": 1000
      },
      "aggs": {
        "systems_sequence": {
          "terms": {
            "field": "system.keyword",
            "order": {
              "_key": "asc"
            }
          },
          "aggs": {
            "first_access": {
              "min": {
                "field": "@timestamp"
              }
            },
            "last_access": {
              "max": {
                "field": "@timestamp"
              }
            }
          }
        },
        "total_duration": {
          "bucket_script": {
            "buckets_path": {
              "first": "systems_sequence>first_access",
              "last": "systems_sequence>last_access"
            },
            "script": "params.last - params.first"
          }
        }
      }
    }
  }
}

// Performance Metrics Aggregation
{
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "log_message": "requestUri"
          }
        },
        {
          "range": {
            "@timestamp": {
              "gte": "now-1h"
            }
          }
        }
      ]
    }
  },
  "aggs": {
    "performance_by_system": {
      "terms": {
        "field": "system.keyword"
      },
      "aggs": {
        "request_timeline": {
          "date_histogram": {
            "field": "@timestamp",
            "fixed_interval": "1m"
          }
        },
        "modules_accessed": {
          "terms": {
            "field": "module.keyword",
            "size": 20
          },
          "aggs": {
            "access_count": {
              "value_count": {
                "field": "module.keyword"
              }
            }
          }
        }
      }
    }
  }
}
```

---

## 2. Database Schema Specifications (Elasticsearch Mapping)

### 2.1 Activity Logs Index Mapping
```json
{
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "index": {
      "refresh_interval": "5s",
      "max_result_window": 50000
    }
  },
  "mappings": {
    "properties": {
      "@timestamp": {
        "type": "date",
        "format": "strict_date_optional_time||epoch_millis"
      },
      "session_id": {
        "type": "keyword",
        "index": true
      },
      "session_token": {
        "type": "keyword",
        "index": true
      },
      "session_number": {
        "type": "long"
      },
      "session_timestamp": {
        "type": "long"
      },
      "username": {
        "type": "keyword",
        "index": true,
        "fields": {
          "text": {
            "type": "text",
            "analyzer": "standard"
          }
        }
      },
      "system": {
        "type": "keyword",
        "index": true
      },
      "module": {
        "type": "keyword", 
        "index": true
      },
      "page": {
        "type": "keyword",
        "index": true
      },
      "activity_type": {
        "type": "keyword",
        "index": true
      },
      "level": {
        "type": "keyword",
        "index": true
      },
      "class_info": {
        "type": "text",
        "analyzer": "standard",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "log_message": {
        "type": "text",
        "analyzer": "standard"
      },
      "log_source": {
        "type": "keyword",
        "index": true
      },
      "request_uri": {
        "type": "keyword",
        "index": true
      },
      "ip_address": {
        "type": "ip"
      },
      "user_agent": {
        "type": "text",
        "analyzer": "standard"
      },
      "response_time": {
        "type": "float"
      },
      "request_method": {
        "type": "keyword"
      },
      "status_code": {
        "type": "integer"
      },
      "geographic_info": {
        "properties": {
          "country": {
            "type": "keyword"
          },
          "region": {
            "type": "keyword"
          },
          "city": {
            "type": "keyword"
          },
          "location": {
            "type": "geo_point"
          }
        }
      },
      "security_context": {
        "properties": {
          "roles": {
            "type": "keyword"
          },
          "permissions": {
            "type": "keyword"
          },
          "security_level": {
            "type": "keyword"
          }
        }
      }
    }
  }
}
```

### 2.2 User Sessions Index Mapping
```json
{
  "settings": {
    "number_of_shards": 2,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "session_id": {
        "type": "keyword",
        "index": true
      },
      "username": {
        "type": "keyword",
        "index": true
      },
      "start_time": {
        "type": "date"
      },
      "end_time": {
        "type": "date"
      },
      "duration_minutes": {
        "type": "float"
      },
      "systems_accessed": {
        "type": "keyword"
      },
      "modules_accessed": {
        "type": "keyword"
      },
      "total_requests": {
        "type": "integer"
      },
      "ip_address": {
        "type": "ip"
      },
      "status": {
        "type": "keyword"
      },
      "last_activity": {
        "type": "date"
      },
      "cross_system_navigation": {
        "type": "nested",
        "properties": {
          "from_system": {
            "type": "keyword"
          },
          "to_system": {
            "type": "keyword"
          },
          "timestamp": {
            "type": "date"
          },
          "transition_time": {
            "type": "float"
          }
        }
      }
    }
  }
}
```

### 2.3 System Performance Index Mapping
```json
{
  "settings": {
    "number_of_shards": 1,
    "number_of_replicas": 1
  },
  "mappings": {
    "properties": {
      "@timestamp": {
        "type": "date"
      },
      "system": {
        "type": "keyword"
      },
      "metric_type": {
        "type": "keyword"
      },
      "response_time_avg": {
        "type": "float"
      },
      "response_time_p95": {
        "type": "float"
      },
      "response_time_p99": {
        "type": "float"
      },
      "request_count": {
        "type": "integer"
      },
      "error_count": {
        "type": "integer"
      },
      "error_rate": {
        "type": "float"
      },
      "active_users": {
        "type": "integer"
      },
      "memory_usage": {
        "properties": {
          "heap_total": {
            "type": "float"
          },
          "heap_used": {
            "type": "float"
          },
          "heap_percentage": {
            "type": "float"
          }
        }
      },
      "cpu_usage": {
        "type": "float"
      },
      "disk_usage": {
        "type": "float"
      }
    }
  }
}
```

---

## 3. Component Specifications

### 3.1 JSF Managed Bean Specifications

#### 3.1.1 Main Dashboard Bean
```java
@Named("dashboardBean")
@ViewScoped
public class DashboardBean implements Serializable {
    
    private static final long serialVersionUID = 1L;
    
    @Inject
    private LogAnalysisService logAnalysisService;
    
    @Inject
    private ElasticsearchService elasticsearchService;
    
    @Inject
    private SecurityService securityService;
    
    // Dashboard Model for PrimeFaces Dashboard Component
    private DashboardModel dashboardModel;
    
    // Real-time Data Properties
    private int activeUsers;
    private int totalSessions; 
    private List<ActivitySummary> recentActivities;
    private LineChartModel authTrendChart;
    private DonutChartModel systemUsageChart;
    private List<SystemHealthStatus> systemHealthList;
    
    // Filter Properties
    private Date fromDate;
    private Date toDate;
    private String selectedSystem;
    private String selectedUser;
    
    @PostConstruct
    public void init() {
        // Initialize security context
        securityService.validateUserAccess();
        
        // Initialize date range (default: last 24 hours)
        toDate = new Date();
        fromDate = DateUtils.addDays(toDate, -1);
        
        // Create dashboard layout
        createDashboardModel();
        
        // Load initial data
        refreshDashboardData();
        
        // Initialize charts
        createAuthTrendChart();
        createSystemUsageChart();
    }
    
    /**
     * Creates PrimeFaces dashboard model with customizable panels
     */
    private void createDashboardModel() {
        dashboardModel = new DefaultDashboardModel();
        DashboardColumn column1 = new DefaultDashboardColumn();
        DashboardColumn column2 = new DefaultDashboardColumn();
        DashboardColumn column3 = new DefaultDashboardColumn();
        
        // Authentication Panel
        column1.addWidget("authPanel");
        
        // System Usage Panel  
        column1.addWidget("systemPanel");
        
        // Performance Panel
        column2.addWidget("performancePanel");
        
        // Recent Activities Panel
        column2.addWidget("activitiesPanel");
        
        // Alerts Panel
        column3.addWidget("alertsPanel");
        
        // System Health Panel
        column3.addWidget("healthPanel");
        
        dashboardModel.addColumn(column1);
        dashboardModel.addColumn(column2);
        dashboardModel.addColumn(column3);
    }
    
    /**
     * Refreshes dashboard data from Elasticsearch
     */
    public void refreshDashboardData() {
        try {
            // Get current statistics
            DashboardStatistics stats = logAnalysisService.getDashboardStatistics(fromDate, toDate);
            this.activeUsers = stats.getActiveUsers();
            this.totalSessions = stats.getTotalSessions();
            
            // Get recent activities
            this.recentActivities = logAnalysisService.getRecentActivities(50);
            
            // Get system health
            this.systemHealthList = logAnalysisService.getSystemHealthStatus();
            
            // Update charts
            updateCharts();
            
            // Show success message
            FacesContext.getCurrentInstance().addMessage(null, 
                new FacesMessage(FacesMessage.SEVERITY_INFO, "Success", "Dashboard refreshed successfully"));
                
        } catch (Exception e) {
            // Log error
            Logger.getLogger(DashboardBean.class.getName()).log(Level.SEVERE, "Error refreshing dashboard", e);
            
            // Show error message
            FacesContext.getCurrentInstance().addMessage(null,
                new FacesMessage(FacesMessage.SEVERITY_ERROR, "Error", "Failed to refresh dashboard: " + e.getMessage()));
        }
    }
    
    /**
     * Auto-refresh method called by p:poll component
     */
    public void autoRefresh() {
        refreshDashboardData();
    }
    
    /**
     * Creates authentication trend chart using Chart.js through PrimeFaces
     */
    private void createAuthTrendChart() {
        authTrendChart = new LineChartModel();
        
        LineChartSeries loginSeries = new LineChartSeries();
        loginSeries.setLabel("Logins");
        
        LineChartSeries logoutSeries = new LineChartSeries();
        logoutSeries.setLabel("Logouts");
        
        // Get hourly login/logout data from Elasticsearch
        List<HourlyAuthStats> authStats = logAnalysisService.getHourlyAuthStats(fromDate, toDate);
        
        for (HourlyAuthStats stat : authStats) {
            loginSeries.set(stat.getHour(), stat.getLoginCount());
            logoutSeries.set(stat.getHour(), stat.getLogoutCount());
        }
        
        authTrendChart.addSeries(loginSeries);
        authTrendChart.addSeries(logoutSeries);
        
        // Chart options
        authTrendChart.setTitle("Authentication Trends");
        authTrendChart.setLegendPosition("nw");
        authTrendChart.setAnimate(true);
        authTrendChart.setShowPointLabels(true);
        
        Axis xAxis = authTrendChart.getAxis(AxisType.X);
        xAxis.setLabel("Hour");
        
        Axis yAxis = authTrendChart.getAxis(AxisType.Y);
        yAxis.setLabel("Count");
        yAxis.setMin(0);
    }
    
    /**
     * Creates system usage donut chart
     */
    private void createSystemUsageChart() {
        systemUsageChart = new DonutChartModel();
        
        // Get system usage data
        Map<String, Integer> systemUsage = logAnalysisService.getSystemUsageStats(fromDate, toDate);
        
        for (Map.Entry<String, Integer> entry : systemUsage.entrySet()) {
            systemUsageChart.set(entry.getKey().toUpperCase(), entry.getValue());
        }
        
        systemUsageChart.setTitle("System Usage Distribution");
        systemUsageChart.setLegendPosition("w");
        systemUsageChart.setShowDataLabels(true);
        systemUsageChart.setDataFormat("value");
    }
    
    /**
     * Updates all charts with latest data
     */
    private void updateCharts() {
        createAuthTrendChart();
        createSystemUsageChart();
    }
    
    /**
     * Applies date range filter
     */
    public void applyDateFilter() {
        refreshDashboardData();
    }
    
    /**
     * Exports dashboard data to Excel
     */
    public void exportToExcel() {
        try {
            // Implementation using Apache POI
            ExcelExportService excelService = new ExcelExportService();
            Workbook workbook = excelService.createDashboardExport(
                recentActivities, systemHealthList, fromDate, toDate);
            
            // Prepare response
            FacesContext facesContext = FacesContext.getCurrentInstance();
            ExternalContext externalContext = facesContext.getExternalContext();
            HttpServletResponse response = (HttpServletResponse) externalContext.getResponse();
            
            response.setContentType("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet");
            response.setHeader("Content-Disposition", "attachment; filename=\"dashboard-export.xlsx\"");
            
            workbook.write(response.getOutputStream());
            facesContext.responseComplete();
            
        } catch (Exception e) {
            Logger.getLogger(DashboardBean.class.getName()).log(Level.SEVERE, "Error exporting to Excel", e);
            FacesContext.getCurrentInstance().addMessage(null,
                new FacesMessage(FacesMessage.SEVERITY_ERROR, "Error", "Failed to export data"));
        }
    }
    
    // Getters and Setters
    public DashboardModel getDashboardModel() { return dashboardModel; }
    public void setDashboardModel(DashboardModel dashboardModel) { this.dashboardModel = dashboardModel; }
    
    public int getActiveUsers() { return activeUsers; }
    public int getTotalSessions() { return totalSessions; }
    
    public List<ActivitySummary> getRecentActivities() { return recentActivities; }
    
    public LineChartModel getAuthTrendChart() { return authTrendChart; }
    public DonutChartModel getSystemUsageChart() { return systemUsageChart; }
    
    public List<SystemHealthStatus> getSystemHealthList() { return systemHealthList; }
    
    public Date getFromDate() { return fromDate; }
    public void setFromDate(Date fromDate) { this.fromDate = fromDate; }
    
    public Date getToDate() { return toDate; }
    public void setToDate(Date toDate) { this.toDate = toDate; }
    
    public String getSelectedSystem() { return selectedSystem; }
    public void setSelectedSystem(String selectedSystem) { this.selectedSystem = selectedSystem; }
    
    public String getSelectedUser() { return selectedUser; }
    public void setSelectedUser(String selectedUser) { this.selectedUser = selectedUser; }
}
```

#### 3.1.2 Log Analysis Service Bean
```java
@Stateless
@LocalBean
public class LogAnalysisService {
    
    @Inject
    private ElasticsearchService elasticsearchService;
    
    @Inject
    private LogParserService logParserService;
    
    @Inject
    private SecurityService securityService;
    
    // Production Log Pattern Matchers
    private static final Pattern SESSION_PATTERN = 
        Pattern.compile("\\[([a-zA-Z0-9!_-]{50,70})\\]");
    private static final Pattern USER_PATTERN = 
        Pattern.compile("\\]\\[([^\\]\\(]+)(?:\\([^\\)]+\\))?\\]\\[");
    private static final Pattern MODULE_PATTERN = 
        Pattern.compile("/(edcs|leav|ofc-app)/pages/(?:[^/]+/)*([^/]+)/");
    private static final Pattern REQUEST_URI_PATTERN = 
        Pattern.compile("requestUri = ([^\\s]+)");
    private static final Pattern LOGIN_PATTERN = 
        Pattern.compile("\\[([^\\]]+)\\(login\\)\\]");
    
    /**
     * Analyzes recent log entries and returns activity summaries
     */
    public List<ActivitySummary> getRecentActivities(int maxResults) {
        List<ActivitySummary> activities = new ArrayList<>();
        
        try {
            // Build Elasticsearch query for recent activities
            SearchRequest searchRequest = new SearchRequest("activity-logs-*");
            SearchSourceBuilder searchSourceBuilder = new SearchSourceBuilder();
            
            BoolQueryBuilder boolQuery = QueryBuilders.boolQuery()
                .must(QueryBuilders.rangeQuery("@timestamp")
                    .gte("now-1h")
                    .lte("now"))
                .mustNot(QueryBuilders.matchQuery("level", "TRACE")); // Exclude trace logs
            
            searchSourceBuilder.query(boolQuery);
            searchSourceBuilder.sort("@timestamp", SortOrder.DESC);
            searchSourceBuilder.size(maxResults);
            
            searchRequest.source(searchSourceBuilder);
            
            SearchResponse searchResponse = elasticsearchService.search(searchRequest);
            
            // Process search hits
            for (SearchHit hit : searchResponse.getHits().getHits()) {
                Map<String, Object> sourceMap = hit.getSourceAsMap();
                ActivitySummary activity = mapToActivitySummary(sourceMap);
                if (activity != null) {
                    activities.add(activity);
                }
            }
            
        } catch (Exception e) {
            Logger.getLogger(LogAnalysisService.class.getName())
                  .log(Level.SEVERE, "Error analyzing recent activities", e);
        }
        
        return activities;
    }
    
    /**
     * Gets dashboard statistics for specified date range
     */
    public DashboardStatistics getDashboardStatistics(Date fromDate, Date toDate) {
        DashboardStatistics stats = new DashboardStatistics();
        
        try {
            // Query for active sessions
            SearchRequest sessionRequest = new SearchRequest("activity-logs-*");
            SearchSourceBuilder sessionBuilder = new SearchSourceBuilder();
            
            BoolQueryBuilder sessionQuery = QueryBuilders.boolQuery()
                .must(QueryBuilders.rangeQuery("@timestamp")
                    .gte(fromDate.getTime())
                    .lte(toDate.getTime()));
            
            // Aggregation for unique sessions
            CardinalityAggregationBuilder sessionAgg = 
                AggregationBuilders.cardinality("unique_sessions")
                    .field("session_id.keyword");
            
            // Aggregation for unique users
            CardinalityAggregationBuilder userAgg = 
                AggregationBuilders.cardinality("unique_users")
                    .field("username.keyword");
            
            sessionBuilder.query(sessionQuery);
            sessionBuilder.aggregation(sessionAgg);
            sessionBuilder.aggregation(userAgg);
            sessionBuilder.size(0); // Only aggregations, no hits
            
            sessionRequest.source(sessionBuilder);
            
            SearchResponse sessionResponse = elasticsearchService.search(sessionRequest);
            
            // Extract aggregation results
            Cardinality sessionCount = sessionResponse.getAggregations().get("unique_sessions");
            Cardinality userCount = sessionResponse.getAggregations().get("unique_users");
            
            stats.setTotalSessions((int) sessionCount.getValue());
            stats.setActiveUsers((int) userCount.getValue());
            
            // Calculate additional metrics
            stats.setAverageSessionDuration(calculateAverageSessionDuration(fromDate, toDate));
            stats.setSystemLoad(calculateSystemLoad());
            
        } catch (Exception e) {
            Logger.getLogger(LogAnalysisService.class.getName())
                  .log(Level.SEVERE, "Error getting dashboard statistics", e);
        }
        
        return stats;
    }
    
    /**
     * Gets hourly authentication statistics
     */
    public List<HourlyAuthStats> getHourlyAuthStats(Date fromDate, Date toDate) {
        List<HourlyAuthStats> hourlyStats = new ArrayList<>();
        
        try {
            SearchRequest request = new SearchRequest("activity-logs-*");
            SearchSourceBuilder builder = new SearchSourceBuilder();
            
            // Query for login activities
            BoolQueryBuilder query = QueryBuilders.boolQuery()
                .must(QueryBuilders.rangeQuery("@timestamp")
                    .gte(fromDate.getTime())
                    .lte(toDate.getTime()))
                .should(QueryBuilders.matchQuery("activity_type", "login"))
                .should(QueryBuilders.matchQuery("log_message", "login"));
            
            // Date histogram aggregation
            DateHistogramAggregationBuilder dateHistogram = 
                AggregationBuilders.dateHistogram("hourly_auth")
                    .field("@timestamp")
                    .fixedInterval(DateHistogramInterval.HOUR)
                    .format("yyyy-MM-dd HH:mm:ss");
            
            // Sub-aggregation for login count
            TermsAggregationBuilder activityTerms = 
                AggregationBuilders.terms("activity_types")
                    .field("activity_type.keyword");
            
            dateHistogram.subAggregation(activityTerms);
            
            builder.query(query);
            builder.aggregation(dateHistogram);
            builder.size(0);
            
            request.source(builder);
            
            SearchResponse response = elasticsearchService.search(request);
            
            // Process aggregation results
            Histogram dateHistogramAgg = response.getAggregations().get("hourly_auth");
            
            for (Histogram.Bucket bucket : dateHistogramAgg.getBuckets()) {
                HourlyAuthStats stats = new HourlyAuthStats();
                stats.setHour(bucket.getKeyAsString());
                stats.setLoginCount((int) bucket.getDocCount());
                
                // Process sub-aggregations for activity breakdown
                Terms activityTermsAgg = bucket.getAggregations().get("activity_types");
                for (Terms.Bucket activityBucket : activityTermsAgg.getBuckets()) {
                    String activityType = activityBucket.getKeyAsString();
                    int count = (int) activityBucket.getDocCount();
                    
                    if ("login".equals(activityType)) {
                        stats.setLoginCount(count);
                    } else if ("logout".equals(activityType)) {
                        stats.setLogoutCount(count);
                    }
                }
                
                hourlyStats.add(stats);
            }
            
        } catch (Exception e) {
            Logger.getLogger(LogAnalysisService.class.getName())
                  .log(Level.SEVERE, "Error getting hourly auth stats", e);
        }
        
        return hourlyStats;
    }
    
    /**
     * Gets system usage statistics
     */
    public Map<String, Integer> getSystemUsageStats(Date fromDate, Date toDate) {
        Map<String, Integer> systemUsage = new HashMap<>();
        
        try {
            SearchRequest request = new SearchRequest("activity-logs-*");
            SearchSourceBuilder builder = new SearchSourceBuilder();
            
            BoolQueryBuilder query = QueryBuilders.boolQuery()
                .must(QueryBuilders.rangeQuery("@timestamp")
                    .gte(fromDate.getTime())
                    .lte(toDate.getTime()))
                .must(QueryBuilders.existsQuery("system"));
            
            TermsAggregationBuilder systemTerms = 
                AggregationBuilders.terms("system_usage")
                    .field("system.keyword")
                    .size(10);
            
            builder.query(query);
            builder.aggregation(systemTerms);
            builder.size(0);
            
            request.source(builder);
            
            SearchResponse response = elasticsearchService.search(request);
            
            Terms systemTermsAgg = response.getAggregations().get("system_usage");
            
            for (Terms.Bucket bucket : systemTermsAgg.getBuckets()) {
                systemUsage.put(bucket.getKeyAsString(), (int) bucket.getDocCount());
            }
            
        } catch (Exception e) {
            Logger.getLogger(LogAnalysisService.class.getName())
                  .log(Level.SEVERE, "Error getting system usage stats", e);
        }
        
        return systemUsage;
    }
    
    /**
     * Gets system health status for all systems
     */
    public List<SystemHealthStatus> getSystemHealthStatus() {
        List<SystemHealthStatus> healthStatusList = new ArrayList<>();
        
        String[] systems = {"edcs", "leav", "ofc-app"};
        
        for (String system : systems) {
            SystemHealthStatus status = new SystemHealthStatus();
            status.setSystemName(system.toUpperCase());
            
            try {
                // Get latest log entry timestamp
                Date lastLogTime = getLatestLogTimestamp(system);
                status.setLastActivity(lastLogTime);
                
                // Calculate health status based on last activity
                long timeDiff = System.currentTimeMillis() - lastLogTime.getTime();
                long minutesDiff = timeDiff / (1000 * 60);
                
                if (minutesDiff < 5) {
                    status.setStatus("HEALTHY");
                    status.setStatusColor("green");
                } else if (minutesDiff < 15) {
                    status.setStatus("WARNING");
                    status.setStatusColor("orange");
                } else {
                    status.setStatus("CRITICAL");
                    status.setStatusColor("red");
                }
                
                // Get additional metrics
                status.setActiveUsers(getActiveUserCount(system));
                status.setResponseTime(getAverageResponseTime(system));
                status.setErrorRate(getErrorRate(system));
                
            } catch (Exception e) {
                status.setStatus("ERROR");
                status.setStatusColor("red");
                Logger.getLogger(LogAnalysisService.class.getName())
                      .log(Level.WARNING, "Error getting health status for system: " + system, e);
            }
            
            healthStatusList.add(status);
        }
        
        return healthStatusList;
    }
    
    // Helper Methods
    
    private ActivitySummary mapToActivitySummary(Map<String, Object> sourceMap) {
        try {
            ActivitySummary activity = new ActivitySummary();
            
            activity.setTimestamp(parseTimestamp(sourceMap.get("@timestamp")));
            activity.setUsername((String) sourceMap.get("username"));
            activity.setSystem((String) sourceMap.get("system"));
            activity.setModule((String) sourceMap.get("module"));
            activity.setPage((String) sourceMap.get("page"));
            activity.setActivityType((String) sourceMap.get("activity_type"));
            activity.setSessionId((String) sourceMap.get("session_id"));
            activity.setLogMessage((String) sourceMap.get("log_message"));
            
            return activity;
            
        } catch (Exception e) {
            Logger.getLogger(LogAnalysisService.class.getName())
                  .log(Level.WARNING, "Error mapping activity summary", e);
            return null;
        }
    }
    
    private double calculateAverageSessionDuration(Date fromDate, Date toDate) {
        // Implementation to calculate average session duration from logs
        return 25.3; // Placeholder
    }
    
    private double calculateSystemLoad() {
        // Implementation to calculate current system load
        return 78.5; // Placeholder
    }
    
    private Date getLatestLogTimestamp(String system) {
        // Implementation to get latest log entry timestamp for system
        return new Date(); // Placeholder
    }
    
    private int getActiveUserCount(String system) {
        // Implementation to get active user count for system
        return 45; // Placeholder
    }
    
    private String getAverageResponseTime(String system) {
        // Implementation to get average response time for system
        return "1.2s"; // Placeholder
    }
    
    private String getErrorRate(String system) {
        // Implementation to get error rate for system
        return "0.02%"; // Placeholder
    }
    
    private Date parseTimestamp(Object timestamp) {
        if (timestamp instanceof String) {
            try {
                return new SimpleDateFormat("yyyy-MM-dd'T'HH:mm:ss.SSS'Z'").parse((String) timestamp);
            } catch (ParseException e) {
                return new Date();
            }
        }
        return new Date();
    }
}
```

---

## 4. Integration Specifications

### 4.1 Logstash Pipeline Configuration

#### 4.1.1 Production Log Processing Pipeline
```ruby
# /etc/logstash/pipelines.yml
- pipeline.id: activity-tracking-production
  path.config: "/usr/share/logstash/pipeline/production-logs.conf"
  pipeline.workers: 4
  pipeline.batch.size: 1000
  pipeline.batch.delay: 5

# /usr/share/logstash/pipeline/production-logs.conf
input {
  # EDCS System Logs
  file {
    path => "/var/log/applications/edcs.log"
    type => "edcs"
    start_position => "end"
    sincedb_path => "/var/lib/logstash/sincedb_edcs"
    codec => multiline {
      pattern => "^%{TIMESTAMP_ISO8601}"
      negate => true
      what => "previous"
    }
  }
  
  # LEAV System Logs  
  file {
    path => "/var/log/applications/leav.log"
    type => "leav"
    start_position => "end"
    sincedb_path => "/var/lib/logstash/sincedb_leav"
    codec => multiline {
      pattern => "^%{TIMESTAMP_ISO8601}"
      negate => true
      what => "previous"
    }
  }
  
  # OFC-APP System Logs
  file {
    path => "/var/log/applications/ofc-app.log"
    type => "ofc-app"
    start_position => "end"
    sincedb_path => "/var/lib/logstash/sincedb_ofc_app"
    codec => multiline {
      pattern => "^%{TIMESTAMP_ISO8601}"
      negate => true
      what => "previous"
    }
  }
}

filter {
  # Parse main log pattern
  grok {
    match => { 
      "message" => "%{TIME:log_time} %{LOGLEVEL:level} \[%{DATA:session_id}\]\[%{DATA:username}\]\[%{DATA:class_info}\] %{GREEDYDATA:log_message}"
    }
    tag_on_failure => ["_grokparsefailure_main"]
  }
  
  # Extract complex session ID components
  if [session_id] {
    grok {
      match => { 
        "session_id" => "(?<session_token>[a-zA-Z0-9_-]{40,60})!(?<session_number>\d+)!(?<session_timestamp>\d+)"
      }
      tag_on_failure => ["_grokparsefailure_session"]
    }
  }
  
  # Clean username (remove action context like "(login)")
  if [username] and [username] != "-" {
    if [username] =~ /\(login\)/ {
      mutate {
        add_field => { "activity_type" => "login" }
        gsub => [ "username", "\(login\)", "" ]
      }
    }
    if [username] =~ /\(logout\)/ {
      mutate {
        add_field => { "activity_type" => "logout" }  
        gsub => [ "username", "\(logout\)", "" ]
      }
    }
  }
  
  # Extract request URI and module information
  if [log_message] =~ /requestUri/ {
    grok {
      match => { 
        "log_message" => "requestUri = /(?<system>edcs|leav|ofc-app)/pages/(?<module_path>[^\\s\\?]+)"
      }
      tag_on_failure => ["_grokparsefailure_uri"]
    }
    
    # Extract module name from path
    if [module_path] {
      grok {
        match => {
          "module_path" => "(?:[^/]+/)*(?<module>[^/]+)/(?<page>[^/\\?]+)"
        }
        tag_on_failure => ["_grokparsefailure_module"]
      }
    }
  }
  
  # Extract MDC Filter information (IP address, HTTP method, etc.)
  if [log_message] =~ /MDCFilter/ {
    grok {
      match => {
        "log_message" => "(?<ip_address>\\d+\\.\\d+\\.\\d+\\.\\d+)\\|(?<http_version>[^\\|]+)\\|(?<protocol>[^\\|]+)\\|(?<method>[^\\|]+)\\|(?<request_path>[^\\|]*)\\|\\|(?<full_uri>[^\\|]+)\\|(?<page_path>[^\\s]+)"
      }
      tag_on_failure => ["_grokparsefailure_mdc"]
    }
    
    # Mask IP address for privacy (keep first 3 octets, mask last)
    if [ip_address] {
      mutate {
        gsub => [ "ip_address", "\\.(\\d+)$", ".***" ]
      }
    }
  }
  
  # Extract memory usage information
  if [log_message] =~ /heap/ {
    grok {
      match => {
        "log_message" => "Current heap:(?<heap_total>[\\d\\.]+) MiB; Used:(?<heap_used>[\\d\\.]+) MiB"
      }
      tag_on_failure => ["_grokparsefailure_memory"]
    }
    
    if [heap_total] and [heap_used] {
      ruby {
        code => "
          total = event.get('heap_total').to_f
          used = event.get('heap_used').to_f
          if total > 0
            percentage = (used / total) * 100
            event.set('heap_percentage', percentage.round(2))
          end
        "
      }
    }
  }
  
  # Detect activity types from log content
  if [log_message] {
    if [log_message] =~ /Entering login/ {
      mutate { add_field => { "activity_type" => "login" } }
    }
    if [log_message] =~ /login success/ {
      mutate { add_field => { "activity_type" => "login_success" } }
    }
    if [log_message] =~ /logout/ {
      mutate { add_field => { "activity_type" => "logout" } }
    }
    if [log_message] =~ /search/ {
      mutate { add_field => { "activity_type" => "search" } }
    }
    if [log_message] =~ /approve/ {
      mutate { add_field => { "activity_type" => "approve" } }
    }
    if [log_message] =~ /entry/ {
      mutate { add_field => { "activity_type" => "entry" } }
    }
  }
  
  # Parse timestamp with proper timezone
  date {
    match => [ "log_time", "HH:mm:ss.SSS" ]
    target => "@timestamp"
    timezone => "Asia/Bangkok"
  }
  
  # Add metadata
  mutate {
    add_field => { 
      "log_source" => "%{type}"
      "processing_timestamp" => "%{@timestamp}"
    }
    remove_field => [ "host", "path", "message" ]
  }
  
  # Data validation and cleanup
  if [username] == "-" or [username] == "" {
    mutate { remove_field => [ "username" ] }
  }
  
  if [session_id] == "-" or [session_id] == "" {
    mutate { remove_field => [ "session_id" ] }
  }
}

output {
  # Primary output to Elasticsearch
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "activity-logs-%{+YYYY.MM.dd}"
    template_name => "activity-logs"
    template_pattern => "activity-logs-*"
    template => "/usr/share/logstash/templates/activity-logs-template.json"
    document_id => "%{log_source}_%{@timestamp}_%{[session_id]}"
  }
  
  # Debug output (optional, can be disabled in production)
  if "debug" in [tags] {
    stdout { 
      codec => rubydebug 
    }
  }
  
  # Error handling output
  if "_grokparsefailure" in [tags] {
    elasticsearch {
      hosts => ["elasticsearch:9200"]
      index => "logstash-errors-%{+YYYY.MM.dd}"
    }
  }
}
```

#### 4.1.2 Real-time Analytics Pipeline
```ruby
# /usr/share/logstash/pipeline/realtime-analytics.conf
input {
  elasticsearch {
    hosts => ["elasticsearch:9200"]
    index => "activity-logs-*"
    query => '{
      "query": {
        "range": {
          "@timestamp": {
            "gte": "now-5m"
          }
        }
      }
    }'
    schedule => "*/1 * * * *"  # Run every minute
    codec => "json"
  }
}

filter {
  # Calculate session duration
  if [session_id] and [username] {
    aggregate {
      task_id => "%{session_id}"
      code => "
        map['session_id'] = event.get('session_id')
        map['username'] = event.get('username')
        map['start_time'] ||= event.get('@timestamp')
        map['end_time'] = event.get('@timestamp')
        map['systems'] ||= []
        if event.get('system') and not map['systems'].include?(event.get('system'))
          map['systems'] << event.get('system')
        end
        map['request_count'] ||= 0
        map['request_count'] += 1
      "
      push_previous_map_as_event => true
      timeout => 1800  # 30 minutes
    }
    
    # Calculate duration
    ruby {
      code => "
        start_time = event.get('start_time')
        end_time = event.get('end_time')
        if start_time and end_time
          duration = (Time.parse(end_time) - Time.parse(start_time)) / 60.0  # minutes
          event.set('duration_minutes', duration.round(2))
        end
      "
    }
  }
  
  # Performance metrics calculation
  if [log_message] =~ /response time|execution time/ {
    grok {
      match => {
        "log_message" => "(?:response time|execution time)[^\\d]*(?<response_time_ms>\\d+)"
      }
    }
    
    if [response_time_ms] {
      mutate {
        convert => { "response_time_ms" => "integer" }
      }
    }
  }
}

output {
  # Session analytics index
  if [task_id] {
    elasticsearch {
      hosts => ["elasticsearch:9200"]
      index => "session-analytics-%{+YYYY.MM.dd}"
      document_id => "%{session_id}"
    }
  }
  
  # Performance metrics index
  if [response_time_ms] {
    elasticsearch {
      hosts => ["elasticsearch:9200"]
      index => "performance-metrics-%{+YYYY.MM.dd}"
    }
  }
}
```

### 4.2 Elasticsearch Templates and Settings

#### 4.2.1 Activity Logs Template
```json
{
  "index_patterns": ["activity-logs-*"],
  "settings": {
    "number_of_shards": 3,
    "number_of_replicas": 1,
    "index": {
      "refresh_interval": "5s",
      "max_result_window": 50000,
      "lifecycle": {
        "name": "activity-logs-policy",
        "rollover_alias": "activity-logs"
      }
    },
    "analysis": {
      "analyzer": {
        "log_analyzer": {
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
            "lowercase",
            "stop"
          ]
        }
      }
    }
  },
  "mappings": {
    "dynamic": "strict",
    "properties": {
      "@timestamp": {
        "type": "date",
        "format": "strict_date_optional_time||epoch_millis"
      },
      "log_time": {
        "type": "keyword"
      },
      "level": {
        "type": "keyword",
        "index": true
      },
      "session_id": {
        "type": "keyword",
        "index": true
      },
      "session_token": {
        "type": "keyword",
        "index": true
      },
      "session_number": {
        "type": "long"
      },
      "session_timestamp": {
        "type": "long"
      },
      "username": {
        "type": "keyword",
        "index": true,
        "fields": {
          "text": {
            "type": "text",
            "analyzer": "log_analyzer"
          }
        }
      },
      "class_info": {
        "type": "text",
        "analyzer": "log_analyzer",
        "fields": {
          "keyword": {
            "type": "keyword",
            "ignore_above": 256
          }
        }
      },
      "log_message": {
        "type": "text",
        "analyzer": "log_analyzer"
      },
      "system": {
        "type": "keyword",
        "index": true
      },
      "module": {
        "type": "keyword",
        "index": true
      },
      "module_path": {
        "type": "keyword"
      },
      "page": {
        "type": "keyword",
        "index": true
      },
      "activity_type": {
        "type": "keyword",
        "index": true
      },
      "ip_address": {
        "type": "keyword"
      },
      "http_version": {
        "type": "keyword"
      },
      "protocol": {
        "type": "keyword"
      },
      "method": {
        "type": "keyword"
      },
      "request_path": {
        "type": "keyword"
      },
      "full_uri": {
        "type": "keyword"
      },
      "page_path": {
        "type": "keyword"
      },
      "heap_total": {
        "type": "float"
      },
      "heap_used": {
        "type": "float"
      },
      "heap_percentage": {
        "type": "float"
      },
      "response_time_ms": {
        "type": "integer"
      },
      "log_source": {
        "type": "keyword",
        "index": true
      },
      "processing_timestamp": {
        "type": "date"
      },
      "tags": {
        "type": "keyword"
      }
    }
  }
}
```

#### 4.2.2 Index Lifecycle Policy
```json
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_size": "10GB",
            "max_age": "1d",
            "max_docs": 10000000
          },
          "set_priority": {
            "priority": 100
          }
        }
      },
      "warm": {
        "min_age": "7d",
        "actions": {
          "allocate": {
            "number_of_replicas": 0
          },
          "forcemerge": {
            "max_num_segments": 1
          },
          "set_priority": {
            "priority": 50
          }
        }
      },
      "cold": {
        "min_age": "30d",
        "actions": {
          "allocate": {
            "number_of_replicas": 0
          },
          "set_priority": {
            "priority": 0
          }
        }
      },
      "delete": {
        "min_age": "90d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
```

---

**Document Status:** Technical Specifications Complete  
**Next Phase:** Implementation Planning and Prototype Development  
**Estimated Development Time:** 6-8 weeks  
**Ready for:** Development Team Review and Implementation Planning
