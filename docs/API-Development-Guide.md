# API Development Guide - ACTIVITY-TRACKING System

## Document Information
**Project:** ACTIVITY-TRACKING System  
**Version:** 1.0  
**Date:** December 2024  
**Technology Stack:** Java JDK 1.8, JSF 2.x, Spring Boot, Elasticsearch  
**Target Audience:** Backend Developers

---

## 🏗️ API Architecture Overview

### RESTful API Design Principles
```java
/**
 * Activity Tracking System follows RESTful API design patterns:
 * 
 * 1. Resource-based URLs: /api/v1/logs, /api/v1/analytics
 * 2. HTTP methods: GET, POST, PUT, DELETE, PATCH
 * 3. Stateless operations
 * 4. Consistent response formats
 * 5. Proper HTTP status codes
 * 6. API versioning
 */
```

### API Layer Structure
```
API Layer
├── Controllers (REST Endpoints)
├── DTOs (Data Transfer Objects)
├── Validators (Input Validation)
├── Exception Handlers (Error Responses)
├── Interceptors (Cross-cutting Concerns)
└── Documentation (OpenAPI/Swagger)
```

---

## 🌐 Core API Endpoints

### 1. Log Analysis API

#### Search Production Logs
```java
@RestController
@RequestMapping("/api/v1/logs")
@Validated
@Slf4j
public class LogAnalysisController {

    @Autowired
    private LogAnalysisService logAnalysisService;

    /**
     * Search logs with advanced filtering
     * GET /api/v1/logs/search
     */
    @GetMapping("/search")
    public ResponseEntity<ApiResponse<PagedResponse<LogEntryDto>>> searchLogs(
            @Valid @ModelAttribute LogSearchRequest request,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size,
            @RequestParam(defaultValue = "timestamp") String sortBy,
            @RequestParam(defaultValue = "desc") String sortDirection) {
        
        log.info("Searching logs with criteria: {}", request);
        
        try {
            PagedResponse<LogEntryDto> results = logAnalysisService.searchLogs(
                request, page, size, sortBy, sortDirection);
            
            return ResponseEntity.ok(ApiResponse.success(results));
            
        } catch (InvalidSearchCriteriaException e) {
            log.warn("Invalid search criteria: {}", e.getMessage());
            return ResponseEntity.badRequest()
                .body(ApiResponse.error("Invalid search criteria", e.getMessage()));
                
        } catch (ElasticsearchException e) {
            log.error("Elasticsearch error during log search", e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ApiResponse.error("Search service unavailable", 
                    "Unable to process search request"));
        }
    }

    /**
     * Get authentication statistics
     * GET /api/v1/logs/auth-stats
     */
    @GetMapping("/auth-stats")
    public ResponseEntity<ApiResponse<AuthenticationStatsDto>> getAuthStats(
            @RequestParam @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDate startDate,
            @RequestParam @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDate endDate,
            @RequestParam(required = false) String system) {
        
        log.info("Getting auth stats from {} to {} for system: {}", 
            startDate, endDate, system);
        
        try {
            AuthenticationStatsDto stats = logAnalysisService
                .getAuthenticationStats(startDate, endDate, system);
            
            return ResponseEntity.ok(ApiResponse.success(stats));
            
        } catch (DateRangeException e) {
            return ResponseEntity.badRequest()
                .body(ApiResponse.error("Invalid date range", e.getMessage()));
        }
    }
}
```

#### Log Search Request DTO
```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@JsonIgnoreProperties(ignoreUnknown = true)
public class LogSearchRequest {
    
    @Size(min = 1, max = 100, message = "Username must be between 1 and 100 characters")
    private String username;
    
    @Pattern(regexp = "^(EDCS|LEAV|OFC-APP)$", 
             message = "System must be one of: EDCS, LEAV, OFC-APP")
    private String system;
    
    @Pattern(regexp = "^(LOGIN|LOGOUT|ACCESS|DOWNLOAD|UPLOAD|VIEW|MODIFY)$",
             message = "Invalid action type")
    private String action;
    
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime startTime;
    
    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime endTime;
    
    @Size(max = 200, message = "IP address filter too long")
    private String ipAddress;
    
    @Size(max = 500, message = "Free text search too long")
    private String freeText;
    
    @Min(value = 0, message = "Page must be non-negative")
    private Integer page = 0;
    
    @Min(value = 1, message = "Size must be positive")
    @Max(value = 1000, message = "Size cannot exceed 1000")
    private Integer size = 20;
    
    // Custom validation
    @AssertTrue(message = "End time must be after start time")
    public boolean isValidDateRange() {
        return startTime == null || endTime == null || 
               !endTime.isBefore(startTime);
    }
}
```

### 2. Analytics API

#### Real-time Dashboard Data
```java
@RestController
@RequestMapping("/api/v1/analytics")
@PreAuthorize("hasRole('DASHBOARD_USER')")
@Slf4j
public class AnalyticsController {

    @Autowired
    private AnalyticsService analyticsService;
    
    @Autowired
    private CacheManager cacheManager;

    /**
     * Get real-time dashboard statistics
     * GET /api/v1/analytics/dashboard
     */
    @GetMapping("/dashboard")
    @Cacheable(value = "dashboard-stats", key = "#timeRange", unless = "#result == null")
    public ResponseEntity<ApiResponse<DashboardStatsDto>> getDashboardStats(
            @RequestParam(defaultValue = "24h") String timeRange) {
        
        log.info("Fetching dashboard stats for time range: {}", timeRange);
        
        try {
            DashboardStatsDto stats = analyticsService.getDashboardStats(timeRange);
            return ResponseEntity.ok(ApiResponse.success(stats));
            
        } catch (InvalidTimeRangeException e) {
            return ResponseEntity.badRequest()
                .body(ApiResponse.error("Invalid time range", e.getMessage()));
        }
    }

    /**
     * Get user activity timeline
     * GET /api/v1/analytics/user-timeline/{username}
     */
    @GetMapping("/user-timeline/{username}")
    public ResponseEntity<ApiResponse<UserTimelineDto>> getUserTimeline(
            @PathVariable @Pattern(regexp = "^[a-zA-Z0-9._-]+$") String username,
            @RequestParam @DateTimeFormat(pattern = "yyyy-MM-dd") LocalDate date,
            @RequestParam(defaultValue = "false") boolean includeDetails) {
        
        log.info("Fetching timeline for user: {} on date: {}", username, date);
        
        try {
            UserTimelineDto timeline = analyticsService
                .getUserTimeline(username, date, includeDetails);
            
            return ResponseEntity.ok(ApiResponse.success(timeline));
            
        } catch (UserNotFoundException e) {
            return ResponseEntity.notFound().build();
        }
    }

    /**
     * Generate system performance report
     * POST /api/v1/analytics/performance-report
     */
    @PostMapping("/performance-report")
    @PreAuthorize("hasRole('REPORT_GENERATOR')")
    public ResponseEntity<ApiResponse<ReportGenerationResponse>> generatePerformanceReport(
            @Valid @RequestBody PerformanceReportRequest request) {
        
        log.info("Generating performance report: {}", request);
        
        try {
            ReportGenerationResponse response = analyticsService
                .generatePerformanceReport(request);
            
            return ResponseEntity.accepted()
                .body(ApiResponse.success(response));
                
        } catch (ReportGenerationException e) {
            log.error("Failed to generate performance report", e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ApiResponse.error("Report generation failed", e.getMessage()));
        }
    }
}
```

### 3. Admin API

#### System Management
```java
@RestController
@RequestMapping("/api/v1/admin")
@PreAuthorize("hasRole('SYSTEM_ADMIN')")
@Slf4j
public class AdminController {

    @Autowired
    private SystemManagementService systemService;
    
    @Autowired
    private ElasticsearchService elasticsearchService;

    /**
     * Get system health status
     * GET /api/v1/admin/health
     */
    @GetMapping("/health")
    public ResponseEntity<ApiResponse<SystemHealthDto>> getSystemHealth() {
        
        try {
            SystemHealthDto health = systemService.getSystemHealth();
            
            if (health.getOverallStatus() == HealthStatus.CRITICAL) {
                return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE)
                    .body(ApiResponse.success(health));
            }
            
            return ResponseEntity.ok(ApiResponse.success(health));
            
        } catch (Exception e) {
            log.error("Failed to get system health", e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ApiResponse.error("Health check failed", e.getMessage()));
        }
    }

    /**
     * Reindex Elasticsearch data
     * POST /api/v1/admin/reindex
     */
    @PostMapping("/reindex")
    public ResponseEntity<ApiResponse<ReindexResponse>> reindexData(
            @Valid @RequestBody ReindexRequest request) {
        
        log.info("Starting reindex operation: {}", request);
        
        try {
            ReindexResponse response = elasticsearchService.reindex(request);
            
            return ResponseEntity.accepted()
                .body(ApiResponse.success(response));
                
        } catch (ReindexException e) {
            log.error("Reindex operation failed", e);
            return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR)
                .body(ApiResponse.error("Reindex failed", e.getMessage()));
        }
    }

    /**
     * Manage log processing configuration
     * PUT /api/v1/admin/log-processing-config
     */
    @PutMapping("/log-processing-config")
    public ResponseEntity<ApiResponse<Void>> updateLogProcessingConfig(
            @Valid @RequestBody LogProcessingConfigDto config) {
        
        log.info("Updating log processing configuration: {}", config);
        
        try {
            systemService.updateLogProcessingConfig(config);
            
            return ResponseEntity.ok(ApiResponse.success(null, 
                "Log processing configuration updated successfully"));
                
        } catch (ConfigurationException e) {
            return ResponseEntity.badRequest()
                .body(ApiResponse.error("Invalid configuration", e.getMessage()));
        }
    }
}
```

---

## 📋 Data Transfer Objects (DTOs)

### Response DTOs

#### Standard API Response Wrapper
```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@JsonInclude(JsonInclude.Include.NON_NULL)
public class ApiResponse<T> {
    
    private boolean success;
    private T data;
    private String message;
    private String errorCode;
    private LocalDateTime timestamp;
    private List<ValidationError> validationErrors;
    
    // Success response factory methods
    public static <T> ApiResponse<T> success(T data) {
        return ApiResponse.<T>builder()
            .success(true)
            .data(data)
            .timestamp(LocalDateTime.now())
            .build();
    }
    
    public static <T> ApiResponse<T> success(T data, String message) {
        return ApiResponse.<T>builder()
            .success(true)
            .data(data)
            .message(message)
            .timestamp(LocalDateTime.now())
            .build();
    }
    
    // Error response factory methods
    public static <T> ApiResponse<T> error(String message, String errorCode) {
        return ApiResponse.<T>builder()
            .success(false)
            .message(message)
            .errorCode(errorCode)
            .timestamp(LocalDateTime.now())
            .build();
    }
    
    public static <T> ApiResponse<T> validationError(List<ValidationError> errors) {
        return ApiResponse.<T>builder()
            .success(false)
            .message("Validation failed")
            .errorCode("VALIDATION_ERROR")
            .validationErrors(errors)
            .timestamp(LocalDateTime.now())
            .build();
    }
}
```

#### Paginated Response
```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class PagedResponse<T> {
    
    private List<T> content;
    private int page;
    private int size;
    private long totalElements;
    private int totalPages;
    private boolean first;
    private boolean last;
    private boolean hasNext;
    private boolean hasPrevious;
    
    // Factory method for Elasticsearch results
    public static <T> PagedResponse<T> of(List<T> content, int page, int size, long total) {
        int totalPages = (int) Math.ceil((double) total / size);
        
        return PagedResponse.<T>builder()
            .content(content)
            .page(page)
            .size(size)
            .totalElements(total)
            .totalPages(totalPages)
            .first(page == 0)
            .last(page >= totalPages - 1)
            .hasNext(page < totalPages - 1)
            .hasPrevious(page > 0)
            .build();
    }
}
```

### Domain DTOs

#### Log Entry DTO
```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
@JsonInclude(JsonInclude.Include.NON_NULL)
public class LogEntryDto {
    
    private String id;
    
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss.SSS")
    private LocalDateTime timestamp;
    
    private String system;
    private String username;
    private String sessionId;
    private String action;
    private String ipAddress;
    private String userAgent;
    private String documentId;
    private String details;
    private Duration responseTime;
    private String status;
    
    // Custom JSON serialization for complex fields
    @JsonProperty("responseTimeMs")
    public Long getResponseTimeMs() {
        return responseTime != null ? responseTime.toMillis() : null;
    }
    
    // Security filtering for sensitive data
    @JsonIgnore
    public boolean containsSensitiveData() {
        return details != null && (
            details.contains("password") || 
            details.contains("token") || 
            details.contains("secret")
        );
    }
}
```

#### Dashboard Statistics DTO
```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class DashboardStatsDto {
    
    private SystemOverviewDto systemOverview;
    private List<AuthenticationStatsDto> authenticationStats;
    private List<DocumentAccessStatsDto> documentStats;
    private List<PerformanceMetricDto> performanceMetrics;
    private List<ErrorSummaryDto> errorSummary;
    
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private LocalDateTime lastUpdated;
    
    private String timeRange;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class SystemOverviewDto {
        private long totalUsers;
        private long activeUsers;
        private long totalSessions;
        private long activeSessions;
        private long totalRequests;
        private double averageResponseTime;
        private double errorRate;
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class AuthenticationStatsDto {
        private String system;
        private long successfulLogins;
        private long failedLogins;
        private long logouts;
        private double successRate;
        private List<HourlyStatsDto> hourlyBreakdown;
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class DocumentAccessStatsDto {
        private String documentType;
        private long viewCount;
        private long downloadCount;
        private long uploadCount;
        private long modifyCount;
        private List<String> topUsers;
    }
}
```

#### User Timeline DTO
```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class UserTimelineDto {
    
    private String username;
    private LocalDate date;
    private List<TimelineEventDto> events;
    private UserActivitySummaryDto summary;
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class TimelineEventDto {
        
        @JsonFormat(pattern = "HH:mm:ss")
        private LocalTime time;
        
        private String system;
        private String action;
        private String target;
        private String status;
        private Duration duration;
        private String details;
        
        // Color coding for UI
        @JsonProperty("eventType")
        public String getEventType() {
            if (action.contains("LOGIN")) return "authentication";
            if (action.contains("DOWNLOAD") || action.contains("UPLOAD")) return "file";
            if (action.contains("VIEW")) return "access";
            if (action.contains("MODIFY")) return "modification";
            return "other";
        }
    }
    
    @Data
    @Builder
    @NoArgsConstructor
    @AllArgsConstructor
    public static class UserActivitySummaryDto {
        private int totalEvents;
        private Duration totalActiveTime;
        private List<String> systemsUsed;
        private int documentsAccessed;
        private int filesDownloaded;
        private int filesUploaded;
        private LocalTime firstActivity;
        private LocalTime lastActivity;
    }
}
```

---

## 🔧 Service Layer Implementation

### Log Analysis Service
```java
@Service
@Transactional(readOnly = true)
@Slf4j
public class LogAnalysisService {

    @Autowired
    private ElasticsearchService elasticsearchService;
    
    @Autowired
    private LogPatternMatcher patternMatcher;
    
    @Autowired
    private CacheManager cacheManager;

    /**
     * Search logs with advanced filtering and pagination
     */
    public PagedResponse<LogEntryDto> searchLogs(LogSearchRequest request, 
                                               int page, int size, 
                                               String sortBy, String sortDirection) {
        
        log.info("Searching logs: {}", request);
        
        // Build Elasticsearch query
        SearchRequest searchRequest = buildSearchRequest(request, page, size, sortBy, sortDirection);
        
        try {
            SearchResponse response = elasticsearchService.search(searchRequest);
            List<LogEntryDto> logEntries = parseSearchResponse(response);
            
            long totalHits = response.getHits().getTotalHits().value;
            
            return PagedResponse.of(logEntries, page, size, totalHits);
            
        } catch (IOException e) {
            log.error("Elasticsearch search failed", e);
            throw new ElasticsearchException("Search operation failed", e);
        }
    }

    /**
     * Get authentication statistics for time period
     */
    @Cacheable(value = "auth-stats", key = "#startDate + '-' + #endDate + '-' + #system")
    public AuthenticationStatsDto getAuthenticationStats(LocalDate startDate, 
                                                       LocalDate endDate, 
                                                       String system) {
        
        log.info("Getting auth stats from {} to {} for system: {}", startDate, endDate, system);
        
        validateDateRange(startDate, endDate);
        
        // Build aggregation query
        SearchRequest request = buildAuthStatsQuery(startDate, endDate, system);
        
        try {
            SearchResponse response = elasticsearchService.search(request);
            return parseAuthStatsResponse(response);
            
        } catch (IOException e) {
            log.error("Failed to get authentication stats", e);
            throw new ElasticsearchException("Authentication stats query failed", e);
        }
    }

    /**
     * Build Elasticsearch search request from filter criteria
     */
    private SearchRequest buildSearchRequest(LogSearchRequest request, 
                                           int page, int size,
                                           String sortBy, String sortDirection) {
        
        SearchRequest searchRequest = new SearchRequest("activity-logs-*");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        
        // Build query
        BoolQueryBuilder boolQuery = QueryBuilders.boolQuery();
        
        // Add filters
        if (StringUtils.hasText(request.getUsername())) {
            boolQuery.must(QueryBuilders.matchQuery("username", request.getUsername()));
        }
        
        if (StringUtils.hasText(request.getSystem())) {
            boolQuery.must(QueryBuilders.termQuery("system", request.getSystem()));
        }
        
        if (StringUtils.hasText(request.getAction())) {
            boolQuery.must(QueryBuilders.wildcardQuery("action", "*" + request.getAction() + "*"));
        }
        
        if (request.getStartTime() != null || request.getEndTime() != null) {
            RangeQueryBuilder rangeQuery = QueryBuilders.rangeQuery("timestamp");
            if (request.getStartTime() != null) {
                rangeQuery.gte(request.getStartTime().toString());
            }
            if (request.getEndTime() != null) {
                rangeQuery.lte(request.getEndTime().toString());
            }
            boolQuery.must(rangeQuery);
        }
        
        if (StringUtils.hasText(request.getIpAddress())) {
            boolQuery.must(QueryBuilders.termQuery("ipAddress", request.getIpAddress()));
        }
        
        if (StringUtils.hasText(request.getFreeText())) {
            boolQuery.must(QueryBuilders.multiMatchQuery(request.getFreeText(), 
                "username", "action", "details", "documentId"));
        }
        
        sourceBuilder.query(boolQuery);
        
        // Add pagination
        sourceBuilder.from(page * size);
        sourceBuilder.size(size);
        
        // Add sorting
        SortOrder sortOrder = "desc".equalsIgnoreCase(sortDirection) ? 
            SortOrder.DESC : SortOrder.ASC;
        sourceBuilder.sort(sortBy, sortOrder);
        
        // Exclude sensitive fields
        sourceBuilder.fetchSource(null, new String[]{"rawLog", "sensitive"});
        
        searchRequest.source(sourceBuilder);
        
        return searchRequest;
    }

    /**
     * Parse Elasticsearch search response to DTOs
     */
    private List<LogEntryDto> parseSearchResponse(SearchResponse response) {
        
        return Arrays.stream(response.getHits().getHits())
            .map(hit -> {
                Map<String, Object> source = hit.getSourceAsMap();
                
                return LogEntryDto.builder()
                    .id(hit.getId())
                    .timestamp(parseTimestamp(source.get("timestamp")))
                    .system((String) source.get("system"))
                    .username((String) source.get("username"))
                    .sessionId((String) source.get("sessionId"))
                    .action((String) source.get("action"))
                    .ipAddress((String) source.get("ipAddress"))
                    .userAgent((String) source.get("userAgent"))
                    .documentId((String) source.get("documentId"))
                    .details((String) source.get("details"))
                    .responseTime(parseResponseTime(source.get("responseTime")))
                    .status((String) source.get("status"))
                    .build();
            })
            .collect(Collectors.toList());
    }

    /**
     * Parse timestamp from various formats
     */
    private LocalDateTime parseTimestamp(Object timestamp) {
        if (timestamp instanceof String) {
            try {
                return LocalDateTime.parse((String) timestamp, 
                    DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSS"));
            } catch (DateTimeParseException e) {
                log.warn("Failed to parse timestamp: {}", timestamp);
                return LocalDateTime.now();
            }
        }
        return LocalDateTime.now();
    }

    /**
     * Parse response time duration
     */
    private Duration parseResponseTime(Object responseTime) {
        if (responseTime instanceof Number) {
            return Duration.ofMillis(((Number) responseTime).longValue());
        }
        return null;
    }

    /**
     * Validate date range parameters
     */
    private void validateDateRange(LocalDate startDate, LocalDate endDate) {
        if (startDate != null && endDate != null) {
            if (endDate.isBefore(startDate)) {
                throw new DateRangeException("End date must be after start date");
            }
            
            long daysBetween = ChronoUnit.DAYS.between(startDate, endDate);
            if (daysBetween > 365) {
                throw new DateRangeException("Date range cannot exceed 365 days");
            }
        }
    }
}
```

### Analytics Service
```java
@Service
@Slf4j
public class AnalyticsService {

    @Autowired
    private ElasticsearchService elasticsearchService;
    
    @Autowired
    private RedisTemplate<String, Object> redisTemplate;

    /**
     * Get real-time dashboard statistics
     */
    @Cacheable(value = "dashboard-stats", key = "#timeRange", unless = "#result == null")
    public DashboardStatsDto getDashboardStats(String timeRange) {
        
        log.info("Generating dashboard stats for time range: {}", timeRange);
        
        LocalDateTime endTime = LocalDateTime.now();
        LocalDateTime startTime = calculateStartTime(endTime, timeRange);
        
        try {
            // Run parallel aggregation queries
            CompletableFuture<SystemOverviewDto> systemOverview = 
                CompletableFuture.supplyAsync(() -> getSystemOverview(startTime, endTime));
            
            CompletableFuture<List<AuthenticationStatsDto>> authStats = 
                CompletableFuture.supplyAsync(() -> getAuthStatsForDashboard(startTime, endTime));
            
            CompletableFuture<List<DocumentAccessStatsDto>> docStats = 
                CompletableFuture.supplyAsync(() -> getDocumentStats(startTime, endTime));
            
            CompletableFuture<List<PerformanceMetricDto>> perfMetrics = 
                CompletableFuture.supplyAsync(() -> getPerformanceMetrics(startTime, endTime));
            
            // Wait for all results
            CompletableFuture.allOf(systemOverview, authStats, docStats, perfMetrics).join();
            
            return DashboardStatsDto.builder()
                .systemOverview(systemOverview.get())
                .authenticationStats(authStats.get())
                .documentStats(docStats.get())
                .performanceMetrics(perfMetrics.get())
                .lastUpdated(LocalDateTime.now())
                .timeRange(timeRange)
                .build();
                
        } catch (Exception e) {
            log.error("Failed to generate dashboard stats", e);
            throw new AnalyticsException("Dashboard stats generation failed", e);
        }
    }

    /**
     * Generate user activity timeline
     */
    public UserTimelineDto getUserTimeline(String username, LocalDate date, boolean includeDetails) {
        
        log.info("Generating timeline for user: {} on date: {}", username, date);
        
        LocalDateTime startOfDay = date.atStartOfDay();
        LocalDateTime endOfDay = date.plusDays(1).atStartOfDay();
        
        try {
            SearchRequest request = buildUserTimelineQuery(username, startOfDay, endOfDay, includeDetails);
            SearchResponse response = elasticsearchService.search(request);
            
            List<TimelineEventDto> events = parseTimelineEvents(response);
            UserActivitySummaryDto summary = generateActivitySummary(events, date);
            
            return UserTimelineDto.builder()
                .username(username)
                .date(date)
                .events(events)
                .summary(summary)
                .build();
                
        } catch (IOException e) {
            log.error("Failed to generate user timeline", e);
            throw new AnalyticsException("User timeline generation failed", e);
        }
    }

    /**
     * Calculate start time based on time range string
     */
    private LocalDateTime calculateStartTime(LocalDateTime endTime, String timeRange) {
        switch (timeRange.toLowerCase()) {
            case "1h":
                return endTime.minusHours(1);
            case "6h":
                return endTime.minusHours(6);
            case "24h":
                return endTime.minusHours(24);
            case "7d":
                return endTime.minusDays(7);
            case "30d":
                return endTime.minusDays(30);
            default:
                throw new InvalidTimeRangeException("Invalid time range: " + timeRange);
        }
    }

    /**
     * Get system overview statistics
     */
    private SystemOverviewDto getSystemOverview(LocalDateTime startTime, LocalDateTime endTime) {
        
        try {
            SearchRequest request = new SearchRequest("activity-logs-*");
            SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
            
            // Date range filter
            RangeQueryBuilder dateRange = QueryBuilders.rangeQuery("timestamp")
                .gte(startTime.toString())
                .lte(endTime.toString());
            
            sourceBuilder.query(dateRange);
            sourceBuilder.size(0); // Only aggregations
            
            // Total requests
            sourceBuilder.aggregation(
                AggregationBuilders.valueCount("total_requests").field("timestamp"));
            
            // Unique users
            sourceBuilder.aggregation(
                AggregationBuilders.cardinality("unique_users").field("username.keyword"));
            
            // Unique sessions
            sourceBuilder.aggregation(
                AggregationBuilders.cardinality("unique_sessions").field("sessionId.keyword"));
            
            // Average response time
            sourceBuilder.aggregation(
                AggregationBuilders.avg("avg_response_time").field("responseTime"));
            
            // Error rate
            sourceBuilder.aggregation(
                AggregationBuilders.filter("error_count", 
                    QueryBuilders.rangeQuery("status").gte(400))
                    .subAggregation(AggregationBuilders.valueCount("count").field("status")));
            
            request.source(sourceBuilder);
            SearchResponse response = elasticsearchService.search(request);
            
            Aggregations aggregations = response.getAggregations();
            
            long totalRequests = ((ValueCount) aggregations.get("total_requests")).getValue();
            long uniqueUsers = ((Cardinality) aggregations.get("unique_users")).getValue();
            long uniqueSessions = ((Cardinality) aggregations.get("unique_sessions")).getValue();
            double avgResponseTime = ((Avg) aggregations.get("avg_response_time")).getValue();
            
            Filter errorFilter = aggregations.get("error_count");
            long errorCount = ((ValueCount) errorFilter.getAggregations().get("count")).getValue();
            double errorRate = totalRequests > 0 ? (double) errorCount / totalRequests * 100 : 0;
            
            return SystemOverviewDto.builder()
                .totalUsers(uniqueUsers)
                .activeUsers(uniqueUsers) // Approximate for now
                .totalSessions(uniqueSessions)
                .activeSessions(uniqueSessions) // Approximate for now
                .totalRequests(totalRequests)
                .averageResponseTime(avgResponseTime)
                .errorRate(errorRate)
                .build();
                
        } catch (IOException e) {
            log.error("Failed to get system overview", e);
            throw new AnalyticsException("System overview query failed", e);
        }
    }
}
```

---

## 🔒 Security Implementation

### JWT Authentication
```java
@Component
@Slf4j
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    @Autowired
    private JwtTokenProvider tokenProvider;
    
    @Autowired
    private UserDetailsService userDetailsService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, 
                                  HttpServletResponse response, 
                                  FilterChain filterChain) throws ServletException, IOException {
        
        try {
            String token = extractTokenFromRequest(request);
            
            if (token != null && tokenProvider.validateToken(token)) {
                String username = tokenProvider.getUsernameFromToken(token);
                
                UserDetails userDetails = userDetailsService.loadUserByUsername(username);
                UsernamePasswordAuthenticationToken authentication = 
                    new UsernamePasswordAuthenticationToken(userDetails, null, 
                        userDetails.getAuthorities());
                
                authentication.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authentication);
                
                // Add audit logging
                log.info("User {} authenticated successfully from IP: {}", 
                    username, getClientIpAddress(request));
            }
            
        } catch (Exception e) {
            log.error("Cannot set user authentication", e);
        }
        
        filterChain.doFilter(request, response);
    }

    private String extractTokenFromRequest(HttpServletRequest request) {
        String bearerToken = request.getHeader("Authorization");
        if (StringUtils.hasText(bearerToken) && bearerToken.startsWith("Bearer ")) {
            return bearerToken.substring(7);
        }
        return null;
    }
    
    private String getClientIpAddress(HttpServletRequest request) {
        String xForwardedFor = request.getHeader("X-Forwarded-For");
        if (StringUtils.hasText(xForwardedFor)) {
            return xForwardedFor.split(",")[0].trim();
        }
        
        String xRealIp = request.getHeader("X-Real-IP");
        if (StringUtils.hasText(xRealIp)) {
            return xRealIp;
        }
        
        return request.getRemoteAddr();
    }
}
```

### Role-Based Access Control
```java
@Configuration
@EnableWebSecurity
@EnableGlobalMethodSecurity(prePostEnabled = true)
public class SecurityConfig {

    @Autowired
    private JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;
    
    @Autowired
    private JwtAuthenticationFilter jwtAuthenticationFilter;

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder(12);
    }

    @Bean
    public AuthenticationManager authenticationManager(
            AuthenticationConfiguration config) throws Exception {
        return config.getAuthenticationManager();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf().disable()
            .sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            .and()
            .exceptionHandling()
                .authenticationEntryPoint(jwtAuthenticationEntryPoint)
            .and()
            .authorizeHttpRequests(authz -> authz
                // Public endpoints
                .requestMatchers("/api/v1/auth/**").permitAll()
                .requestMatchers("/actuator/health").permitAll()
                .requestMatchers("/swagger-ui/**", "/v3/api-docs/**").permitAll()
                
                // Dashboard access
                .requestMatchers(HttpMethod.GET, "/api/v1/analytics/dashboard")
                    .hasAnyRole("DASHBOARD_USER", "SYSTEM_ADMIN")
                
                // Log search access
                .requestMatchers(HttpMethod.GET, "/api/v1/logs/**")
                    .hasAnyRole("LOG_VIEWER", "SYSTEM_ADMIN")
                
                // Report generation
                .requestMatchers(HttpMethod.POST, "/api/v1/analytics/reports/**")
                    .hasAnyRole("REPORT_GENERATOR", "SYSTEM_ADMIN")
                
                // Admin endpoints
                .requestMatchers("/api/v1/admin/**")
                    .hasRole("SYSTEM_ADMIN")
                
                // All other requests need authentication
                .anyRequest().authenticated()
            );

        http.addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}
```

---

## 📖 API Documentation

### OpenAPI Configuration
```java
@Configuration
@OpenAPIDefinition(
    info = @Info(
        title = "Activity Tracking System API",
        version = "1.0",
        description = "API for E-Office Activity Tracking and Analytics",
        contact = @Contact(
            name = "E-Office Development Team",
            email = "dev-team@eoffice.local"
        )
    ),
    security = @SecurityRequirement(name = "Bearer Authentication")
)
@SecurityScheme(
    name = "Bearer Authentication",
    type = SecuritySchemeType.HTTP,
    bearerFormat = "JWT",
    scheme = "bearer"
)
public class OpenApiConfig {

    @Bean
    public GroupedOpenApi publicApi() {
        return GroupedOpenApi.builder()
            .group("public")
            .pathsToMatch("/api/v1/**")
            .pathsToExclude("/api/v1/admin/**")
            .build();
    }

    @Bean
    public GroupedOpenApi adminApi() {
        return GroupedOpenApi.builder()
            .group("admin")
            .pathsToMatch("/api/v1/admin/**")
            .build();
    }
}
```

### API Documentation Examples
```java
@RestController
@RequestMapping("/api/v1/logs")
@Tag(name = "Log Analysis", description = "Production log analysis and search operations")
public class LogAnalysisController {

    @Operation(
        summary = "Search production logs",
        description = "Search and filter production logs from EDCS, LEAV, and OFC-APP systems",
        responses = {
            @ApiResponse(
                responseCode = "200",
                description = "Search completed successfully",
                content = @Content(
                    mediaType = "application/json",
                    schema = @Schema(implementation = PagedLogEntryResponse.class),
                    examples = @ExampleObject(
                        name = "Successful search",
                        value = """
                        {
                          "success": true,
                          "data": {
                            "content": [
                              {
                                "id": "abc123",
                                "timestamp": "2024-12-15 14:30:15.123",
                                "system": "EDCS",
                                "username": "sirat",
                                "action": "LOGIN",
                                "ipAddress": "192.168.1.100",
                                "status": "SUCCESS"
                              }
                            ],
                            "page": 0,
                            "size": 20,
                            "totalElements": 1250,
                            "totalPages": 63
                          },
                          "timestamp": "2024-12-15T14:30:20.456"
                        }
                        """
                    )
                )
            ),
            @ApiResponse(
                responseCode = "400",
                description = "Invalid search parameters",
                content = @Content(
                    mediaType = "application/json",
                    schema = @Schema(implementation = ApiResponse.class)
                )
            ),
            @ApiResponse(
                responseCode = "401",
                description = "Authentication required"
            ),
            @ApiResponse(
                responseCode = "403",
                description = "Insufficient permissions"
            )
        }
    )
    @GetMapping("/search")
    public ResponseEntity<ApiResponse<PagedResponse<LogEntryDto>>> searchLogs(
            
            @Parameter(description = "Username to filter by", example = "sirat")
            @RequestParam(required = false) String username,
            
            @Parameter(description = "System to filter by", schema = @Schema(allowableValues = {"EDCS", "LEAV", "OFC-APP"}))
            @RequestParam(required = false) String system,
            
            @Parameter(description = "Start date and time", example = "2024-12-15 00:00:00")
            @RequestParam(required = false) @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime startTime,
            
            @Parameter(description = "End date and time", example = "2024-12-15 23:59:59")
            @RequestParam(required = false) @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss") LocalDateTime endTime,
            
            @Parameter(description = "Page number (0-based)", example = "0")
            @RequestParam(defaultValue = "0") @Min(0) int page,
            
            @Parameter(description = "Page size", example = "20")
            @RequestParam(defaultValue = "20") @Min(1) @Max(1000) int size
    ) {
        // Implementation...
    }
}
```

---

## 🧪 API Testing

### Unit Testing
```java
@ExtendWith(MockitoExtension.class)
class LogAnalysisControllerTest {

    @Mock
    private LogAnalysisService logAnalysisService;

    @InjectMocks
    private LogAnalysisController controller;

    @Test
    void testSearchLogs_Success() {
        // Given
        LogSearchRequest request = LogSearchRequest.builder()
            .username("sirat")
            .system("EDCS")
            .build();
        
        List<LogEntryDto> mockResults = Arrays.asList(
            LogEntryDto.builder()
                .id("test-1")
                .username("sirat")
                .system("EDCS")
                .action("LOGIN")
                .timestamp(LocalDateTime.now())
                .build()
        );
        
        PagedResponse<LogEntryDto> mockResponse = PagedResponse.of(mockResults, 0, 20, 1);
        
        when(logAnalysisService.searchLogs(any(LogSearchRequest.class), eq(0), eq(20), eq("timestamp"), eq("desc")))
            .thenReturn(mockResponse);

        // When
        ResponseEntity<ApiResponse<PagedResponse<LogEntryDto>>> response = 
            controller.searchLogs(request, 0, 20, "timestamp", "desc");

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(response.getBody().isSuccess()).isTrue();
        assertThat(response.getBody().getData().getContent()).hasSize(1);
        
        verify(logAnalysisService).searchLogs(any(LogSearchRequest.class), eq(0), eq(20), eq("timestamp"), eq("desc"));
    }

    @Test
    void testSearchLogs_InvalidCriteria() {
        // Given
        LogSearchRequest request = LogSearchRequest.builder()
            .startTime(LocalDateTime.now())
            .endTime(LocalDateTime.now().minusHours(1)) // Invalid: end before start
            .build();
        
        when(logAnalysisService.searchLogs(any(), anyInt(), anyInt(), anyString(), anyString()))
            .thenThrow(new InvalidSearchCriteriaException("End time must be after start time"));

        // When
        ResponseEntity<ApiResponse<PagedResponse<LogEntryDto>>> response = 
            controller.searchLogs(request, 0, 20, "timestamp", "desc");

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.BAD_REQUEST);
        assertThat(response.getBody().isSuccess()).isFalse();
        assertThat(response.getBody().getMessage()).contains("Invalid search criteria");
    }
}
```

### Integration Testing
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@TestPropertySource(properties = {
    "spring.profiles.active=test",
    "elasticsearch.host=localhost"
})
class LogAnalysisControllerIntegrationTest {

    @Container
    static ElasticsearchContainer elasticsearch = new ElasticsearchContainer("docker.elastic.co/elasticsearch/elasticsearch:7.15.0")
        .withExposedPorts(9200)
        .withEnv("discovery.type", "single-node")
        .withEnv("xpack.security.enabled", "false");

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private ElasticsearchService elasticsearchService;

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("elasticsearch.host", elasticsearch::getHost);
        registry.add("elasticsearch.port", elasticsearch::getFirstMappedPort);
    }

    @BeforeEach
    void setUp() throws IOException {
        // Create test index and data
        createTestData();
    }

    @Test
    @WithMockUser(roles = "LOG_VIEWER")
    void testSearchLogs_Integration() {
        // Given
        String url = "/api/v1/logs/search?username=sirat&system=EDCS";
        
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + generateTestToken());
        HttpEntity<String> entity = new HttpEntity<>(headers);

        // When
        ResponseEntity<String> response = restTemplate.exchange(
            url, HttpMethod.GET, entity, String.class);

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        
        JsonNode responseBody = objectMapper.readTree(response.getBody());
        assertThat(responseBody.get("success").asBoolean()).isTrue();
        assertThat(responseBody.get("data").get("content")).isArray();
    }

    private void createTestData() throws IOException {
        // Insert test log entries into Elasticsearch
        IndexRequest request = new IndexRequest("activity-logs-test")
            .source(XContentType.JSON,
                "timestamp", "2024-12-15T14:30:15.123",
                "system", "EDCS",
                "username", "sirat",
                "action", "LOGIN",
                "ipAddress", "192.168.1.100",
                "status", "SUCCESS"
            );
        
        elasticsearchService.index(request);
        elasticsearchService.refresh("activity-logs-test");
    }
}
```

---

## 🔍 Performance Optimization

### Caching Strategy
```java
@Configuration
@EnableCaching
public class CacheConfig {

    @Bean
    public CacheManager cacheManager(RedisConnectionFactory connectionFactory) {
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
            .entryTtl(Duration.ofMinutes(30))
            .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
            .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()));

        return RedisCacheManager.builder(connectionFactory)
            .cacheDefaults(config)
            .withCacheConfiguration("dashboard-stats", 
                config.entryTtl(Duration.ofMinutes(5)))
            .withCacheConfiguration("auth-stats", 
                config.entryTtl(Duration.ofHours(1)))
            .withCacheConfiguration("user-timeline", 
                config.entryTtl(Duration.ofMinutes(15)))
            .build();
    }
}
```

### Database Query Optimization
```java
@Service
public class OptimizedLogAnalysisService {

    /**
     * Optimized search with index hints and field filtering
     */
    public PagedResponse<LogEntryDto> optimizedSearch(LogSearchRequest request, int page, int size) {
        
        SearchRequest searchRequest = new SearchRequest("activity-logs-*");
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        
        // Use specific index pattern based on date range
        if (request.getStartTime() != null) {
            String indexPattern = buildOptimizedIndexPattern(request.getStartTime(), request.getEndTime());
            searchRequest.indices(indexPattern);
        }
        
        // Build optimized query
        BoolQueryBuilder boolQuery = buildOptimizedQuery(request);
        sourceBuilder.query(boolQuery);
        
        // Fetch only required fields to reduce network overhead
        sourceBuilder.fetchSource(
            new String[]{"timestamp", "system", "username", "action", "ipAddress", "status"},
            new String[]{"rawLog", "details", "userAgent"}
        );
        
        // Add pagination with search_after for large datasets
        if (page > 100) { // Use search_after for deep pagination
            return searchAfterPagination(sourceBuilder, request, page, size);
        } else {
            sourceBuilder.from(page * size);
            sourceBuilder.size(size);
        }
        
        // Add performance optimizations
        sourceBuilder.timeout(TimeValue.timeValueSeconds(10));
        sourceBuilder.terminateAfter(10000); // Prevent very large result sets
        
        try {
            SearchResponse response = elasticsearchService.search(searchRequest);
            return parseOptimizedResponse(response, page, size);
            
        } catch (IOException e) {
            throw new ElasticsearchException("Optimized search failed", e);
        }
    }

    /**
     * Build time-based index pattern for better performance
     */
    private String buildOptimizedIndexPattern(LocalDateTime startTime, LocalDateTime endTime) {
        if (startTime != null && endTime != null) {
            LocalDate startDate = startTime.toLocalDate();
            LocalDate endDate = endTime.toLocalDate();
            
            if (startDate.equals(endDate)) {
                // Single day - use specific daily index
                return "activity-logs-" + startDate.format(DateTimeFormatter.ofPattern("yyyy.MM.dd"));
            } else if (ChronoUnit.DAYS.between(startDate, endDate) <= 7) {
                // Week or less - use specific date range
                return "activity-logs-" + startDate.format(DateTimeFormatter.ofPattern("yyyy.MM")) + "*";
            }
        }
        
        // Default to all indices
        return "activity-logs-*";
    }
}
```

---

**Next:** [Testing Guide](./Testing-Guide.md)  
**See Also:** [Developer Setup Guide](./Developer-Setup-Guide.md) | [Deployment Guide](./Deployment-Guide.md)
