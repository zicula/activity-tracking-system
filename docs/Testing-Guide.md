# Testing Guide - ACTIVITY-TRACKING System

## Document Information
**Project:** ACTIVITY-TRACKING System  
**Version:** 1.0  
**Date:** December 2024  
**Technology Stack:** JUnit 5, Mockito, Testcontainers, Spring Boot Test  
**Target Audience:** QA Engineers, Developers

---

## 🧪 Testing Strategy Overview

### Testing Pyramid
```
                    /\
                   /  \
                  /    \
                 / E2E  \     (10% - End-to-End Tests)
                /________\
               /          \
              / Integration \  (30% - Integration Tests)
             /______________\
            /                \
           /   Unit Tests     \  (60% - Unit Tests)
          /____________________\
```

### Testing Types and Coverage
- **Unit Tests (60%):** Service layer, utilities, DTOs, validators
- **Integration Tests (30%):** API endpoints, database operations, external services
- **End-to-End Tests (10%):** Complete user workflows, dashboard functionality

---

## 🔧 Test Environment Setup

### Maven Dependencies
```xml
<!-- Testing Dependencies -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>junit-jupiter</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.testcontainers</groupId>
    <artifactId>elasticsearch</artifactId>
    <version>1.19.3</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>com.github.tomakehurst</groupId>
    <artifactId>wiremock-jre8</artifactId>
    <version>2.35.0</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.awaitility</groupId>
    <artifactId>awaitility</artifactId>
    <version>4.2.0</version>
    <scope>test</scope>
</dependency>
```

### Test Configuration
```yaml
# application-test.yml
spring:
  profiles:
    active: test
  
  datasource:
    url: jdbc:h2:mem:testdb
    driverClassName: org.h2.Driver
    username: sa
    password: password
  
  jpa:
    hibernate:
      ddl-auto: create-drop
    database-platform: org.hibernate.dialect.H2Dialect

# Elasticsearch test configuration
elasticsearch:
  host: localhost
  port: 9200
  connection-timeout: 5000
  socket-timeout: 30000

# Logging for tests
logging:
  level:
    com.eoffice.activitytracking: DEBUG
    org.springframework.web: DEBUG
    org.elasticsearch: INFO
  pattern:
    console: "%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n"

# Test-specific settings
test:
  elasticsearch:
    cleanup-after-test: true
  data:
    create-sample-data: true
```

---

## 🏗️ Unit Testing

### Service Layer Testing

#### LogAnalysisService Unit Tests
```java
@ExtendWith(MockitoExtension.class)
@DisplayName("LogAnalysisService Unit Tests")
class LogAnalysisServiceTest {

    @Mock
    private ElasticsearchService elasticsearchService;
    
    @Mock
    private LogPatternMatcher patternMatcher;
    
    @Mock
    private CacheManager cacheManager;
    
    @InjectMocks
    private LogAnalysisService logAnalysisService;
    
    @Captor
    private ArgumentCaptor<SearchRequest> searchRequestCaptor;

    @Test
    @DisplayName("Should search logs successfully with valid criteria")
    void testSearchLogs_Success() throws IOException {
        // Given
        LogSearchRequest request = LogSearchRequest.builder()
            .username("sirat")
            .system("EDCS")
            .startTime(LocalDateTime.now().minusHours(1))
            .endTime(LocalDateTime.now())
            .build();
        
        SearchHit[] hits = createMockSearchHits();
        SearchHits searchHits = createMockSearchHits(hits, 100L);
        SearchResponse mockResponse = createMockSearchResponse(searchHits);
        
        when(elasticsearchService.search(any(SearchRequest.class)))
            .thenReturn(mockResponse);

        // When
        PagedResponse<LogEntryDto> result = logAnalysisService.searchLogs(
            request, 0, 20, "timestamp", "desc");

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getContent()).hasSize(2);
        assertThat(result.getTotalElements()).isEqualTo(100L);
        assertThat(result.getPage()).isEqualTo(0);
        assertThat(result.getSize()).isEqualTo(20);
        
        // Verify Elasticsearch query was called
        verify(elasticsearchService).search(searchRequestCaptor.capture());
        SearchRequest capturedRequest = searchRequestCaptor.getValue();
        assertThat(capturedRequest.indices()).contains("activity-logs-*");
        
        // Verify query structure
        SearchSourceBuilder sourceBuilder = capturedRequest.source();
        assertThat(sourceBuilder.size()).isEqualTo(20);
        assertThat(sourceBuilder.from()).isEqualTo(0);
    }

    @Test
    @DisplayName("Should throw exception when Elasticsearch fails")
    void testSearchLogs_ElasticsearchException() throws IOException {
        // Given
        LogSearchRequest request = LogSearchRequest.builder()
            .username("testuser")
            .build();
        
        when(elasticsearchService.search(any(SearchRequest.class)))
            .thenThrow(new IOException("Elasticsearch connection failed"));

        // When & Then
        assertThatThrownBy(() -> logAnalysisService.searchLogs(
            request, 0, 20, "timestamp", "desc"))
            .isInstanceOf(ElasticsearchException.class)
            .hasMessageContaining("Search operation failed");
        
        verify(elasticsearchService).search(any(SearchRequest.class));
    }

    @Test
    @DisplayName("Should validate date range correctly")
    void testSearchLogs_InvalidDateRange() {
        // Given
        LogSearchRequest request = LogSearchRequest.builder()
            .startTime(LocalDateTime.now())
            .endTime(LocalDateTime.now().minusHours(1)) // Invalid: end before start
            .build();

        // When & Then
        assertThatThrownBy(() -> logAnalysisService.searchLogs(
            request, 0, 20, "timestamp", "desc"))
            .isInstanceOf(DateRangeException.class)
            .hasMessageContaining("End date must be after start date");
    }

    @Test
    @DisplayName("Should handle authentication stats request")
    void testGetAuthenticationStats_Success() throws IOException {
        // Given
        LocalDate startDate = LocalDate.now().minusDays(7);
        LocalDate endDate = LocalDate.now();
        String system = "EDCS";
        
        SearchResponse mockResponse = createMockAuthStatsResponse();
        when(elasticsearchService.search(any(SearchRequest.class)))
            .thenReturn(mockResponse);

        // When
        AuthenticationStatsDto result = logAnalysisService
            .getAuthenticationStats(startDate, endDate, system);

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getSystem()).isEqualTo(system);
        assertThat(result.getSuccessfulLogins()).isGreaterThan(0);
        
        verify(elasticsearchService).search(any(SearchRequest.class));
    }

    @Test
    @DisplayName("Should cache authentication stats results")
    void testGetAuthenticationStats_Caching() throws IOException {
        // Given
        LocalDate startDate = LocalDate.now().minusDays(1);
        LocalDate endDate = LocalDate.now();
        String system = "EDCS";
        
        Cache mockCache = mock(Cache.class);
        when(cacheManager.getCache("auth-stats")).thenReturn(mockCache);
        when(mockCache.get(anyString(), eq(AuthenticationStatsDto.class)))
            .thenReturn(null) // First call - cache miss
            .thenReturn(createMockAuthStatsDto()); // Second call - cache hit
        
        SearchResponse mockResponse = createMockAuthStatsResponse();
        when(elasticsearchService.search(any(SearchRequest.class)))
            .thenReturn(mockResponse);

        // When - First call
        AuthenticationStatsDto result1 = logAnalysisService
            .getAuthenticationStats(startDate, endDate, system);
        
        // When - Second call (should use cache)
        AuthenticationStatsDto result2 = logAnalysisService
            .getAuthenticationStats(startDate, endDate, system);

        // Then
        assertThat(result1).isNotNull();
        assertThat(result2).isNotNull();
        
        // Verify Elasticsearch was called only once (first call)
        verify(elasticsearchService, times(1)).search(any(SearchRequest.class));
        verify(mockCache, times(2)).get(anyString(), eq(AuthenticationStatsDto.class));
    }

    // Helper methods for creating mock objects
    private SearchHit[] createMockSearchHits() {
        SearchHit hit1 = mock(SearchHit.class);
        when(hit1.getId()).thenReturn("log-1");
        when(hit1.getSourceAsMap()).thenReturn(Map.of(
            "timestamp", "2024-12-15 14:30:15.123",
            "system", "EDCS",
            "username", "sirat",
            "action", "LOGIN",
            "ipAddress", "192.168.1.100",
            "status", "SUCCESS"
        ));
        
        SearchHit hit2 = mock(SearchHit.class);
        when(hit2.getId()).thenReturn("log-2");
        when(hit2.getSourceAsMap()).thenReturn(Map.of(
            "timestamp", "2024-12-15 14:32:20.456",
            "system", "EDCS",
            "username", "sirat",
            "action", "DOWNLOAD",
            "ipAddress", "192.168.1.100",
            "status", "SUCCESS"
        ));
        
        return new SearchHit[]{hit1, hit2};
    }
    
    private SearchHits createMockSearchHits(SearchHit[] hits, long totalHits) {
        TotalHits total = new TotalHits(totalHits, TotalHits.Relation.EQUAL_TO);
        return new SearchHits(hits, total, 1.0f);
    }
    
    private SearchResponse createMockSearchResponse(SearchHits searchHits) {
        SearchResponse response = mock(SearchResponse.class);
        when(response.getHits()).thenReturn(searchHits);
        return response;
    }
}
```

#### DTO Validation Testing
```java
@DisplayName("DTO Validation Tests")
class DtoValidationTest {

    private Validator validator;

    @BeforeEach
    void setUp() {
        ValidatorFactory factory = Validation.buildDefaultValidatorFactory();
        validator = factory.getValidator();
    }

    @Test
    @DisplayName("LogSearchRequest should validate successfully with valid data")
    void testLogSearchRequest_ValidData() {
        // Given
        LogSearchRequest request = LogSearchRequest.builder()
            .username("validuser")
            .system("EDCS")
            .startTime(LocalDateTime.now().minusHours(1))
            .endTime(LocalDateTime.now())
            .page(0)
            .size(20)
            .build();

        // When
        Set<ConstraintViolation<LogSearchRequest>> violations = validator.validate(request);

        // Then
        assertThat(violations).isEmpty();
    }

    @Test
    @DisplayName("LogSearchRequest should fail validation with invalid system")
    void testLogSearchRequest_InvalidSystem() {
        // Given
        LogSearchRequest request = LogSearchRequest.builder()
            .username("validuser")
            .system("INVALID_SYSTEM")
            .build();

        // When
        Set<ConstraintViolation<LogSearchRequest>> violations = validator.validate(request);

        // Then
        assertThat(violations).hasSize(1);
        assertThat(violations.iterator().next().getMessage())
            .contains("System must be one of: EDCS, LEAV, OFC-APP");
    }

    @Test
    @DisplayName("LogSearchRequest should fail validation with invalid date range")
    void testLogSearchRequest_InvalidDateRange() {
        // Given
        LogSearchRequest request = LogSearchRequest.builder()
            .startTime(LocalDateTime.now())
            .endTime(LocalDateTime.now().minusHours(1)) // End before start
            .build();

        // When
        Set<ConstraintViolation<LogSearchRequest>> violations = validator.validate(request);

        // Then
        assertThat(violations).hasSize(1);
        assertThat(violations.iterator().next().getMessage())
            .contains("End time must be after start time");
    }

    @ParameterizedTest
    @ValueSource(ints = {-1, 1001})
    @DisplayName("LogSearchRequest should fail validation with invalid page size")
    void testLogSearchRequest_InvalidPageSize(int invalidSize) {
        // Given
        LogSearchRequest request = LogSearchRequest.builder()
            .size(invalidSize)
            .build();

        // When
        Set<ConstraintViolation<LogSearchRequest>> violations = validator.validate(request);

        // Then
        assertThat(violations).hasSize(1);
    }
}
```

### Utility Class Testing

#### Log Pattern Matcher Tests
```java
@DisplayName("LogPatternMatcher Unit Tests")
class LogPatternMatcherTest {

    private LogPatternMatcher patternMatcher;

    @BeforeEach
    void setUp() {
        patternMatcher = new LogPatternMatcher();
    }

    @Test
    @DisplayName("Should parse EDCS login log entry correctly")
    void testParseEdcsLoginLog() {
        // Given
        String logLine = "2024-12-15 14:30:15,123 [http-nio-8080-exec-1] INFO  " +
            "c.e.e.s.AuthenticationService - User [sirat] logged in successfully " +
            "from IP [192.168.1.100] with session [sess_abc123def456]";

        // When
        LogEntry result = patternMatcher.parseLogEntry(logLine, "EDCS");

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getTimestamp()).isEqualTo(
            LocalDateTime.of(2024, 12, 15, 14, 30, 15, 123_000_000));
        assertThat(result.getSystem()).isEqualTo("EDCS");
        assertThat(result.getUsername()).isEqualTo("sirat");
        assertThat(result.getAction()).isEqualTo("LOGIN");
        assertThat(result.getIpAddress()).isEqualTo("192.168.1.100");
        assertThat(result.getSessionId()).isEqualTo("sess_abc123def456");
        assertThat(result.getStatus()).isEqualTo("SUCCESS");
    }

    @Test
    @DisplayName("Should parse LEAV document access log correctly")
    void testParseLeavDocumentLog() {
        // Given
        String logLine = "2024-12-15 15:45:30,789 [http-nio-8080-exec-3] INFO  " +
            "c.e.l.s.DocumentService - User [somchai] downloaded document [DOC_20241215_001] " +
            "size [2.5MB] from IP [10.1.1.50] session [sess_xyz789abc123]";

        // When
        LogEntry result = patternMatcher.parseLogEntry(logLine, "LEAV");

        // Then
        assertThat(result).isNotNull();
        assertThat(result.getUsername()).isEqualTo("somchai");
        assertThat(result.getAction()).isEqualTo("DOWNLOAD");
        assertThat(result.getDocumentId()).isEqualTo("DOC_20241215_001");
        assertThat(result.getDetails()).contains("2.5MB");
        assertThat(result.getIpAddress()).isEqualTo("10.1.1.50");
    }

    @ParameterizedTest
    @CsvSource({
        "User [admin] logged in successfully, admin, LOGIN",
        "User [user123] downloaded document, user123, DOWNLOAD",
        "User [test.user] viewed report, test.user, VIEW",
        "User [system_user] modified settings, system_user, MODIFY"
    })
    @DisplayName("Should extract username and action from various log patterns")
    void testExtractUsernameAndAction(String logContent, String expectedUsername, String expectedAction) {
        // Given
        String fullLog = "2024-12-15 10:00:00,000 [thread] INFO class - " + logContent;

        // When
        LogEntry result = patternMatcher.parseLogEntry(fullLog, "TEST");

        // Then
        assertThat(result.getUsername()).isEqualTo(expectedUsername);
        assertThat(result.getAction()).isEqualTo(expectedAction);
    }

    @Test
    @DisplayName("Should handle malformed log entries gracefully")
    void testParseMalformedLog() {
        // Given
        String malformedLog = "This is not a valid log entry format";

        // When & Then
        assertThatThrownBy(() -> patternMatcher.parseLogEntry(malformedLog, "EDCS"))
            .isInstanceOf(LogParsingException.class)
            .hasMessageContaining("Unable to parse log entry");
    }

    @Test
    @DisplayName("Should parse complex session ID patterns")
    void testParseComplexSessionIds() {
        // Given
        Map<String, String> testCases = Map.of(
            "session [sess_abc123def456]", "sess_abc123def456",
            "sessionId=xyz789123abc", "xyz789123abc",
            "SID:12345-67890-ABCDEF", "12345-67890-ABCDEF",
            "session_token_987654321", "session_token_987654321"
        );

        testCases.forEach((logContent, expectedSessionId) -> {
            // When
            String extractedSessionId = patternMatcher.extractSessionId(logContent);

            // Then
            assertThat(extractedSessionId).isEqualTo(expectedSessionId);
        });
    }
}
```

---

## 🔗 Integration Testing

### API Integration Tests

#### LogAnalysisController Integration Tests
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.RANDOM_PORT)
@Testcontainers
@TestPropertySource(properties = {
    "spring.profiles.active=test",
    "elasticsearch.host=localhost"
})
@DisplayName("LogAnalysisController Integration Tests")
class LogAnalysisControllerIntegrationTest {

    @Container
    static ElasticsearchContainer elasticsearch = new ElasticsearchContainer(
        "docker.elastic.co/elasticsearch/elasticsearch:7.15.0")
        .withExposedPorts(9200)
        .withEnv("discovery.type", "single-node")
        .withEnv("xpack.security.enabled", "false")
        .withEnv("ES_JAVA_OPTS", "-Xms512m -Xmx512m");

    @Autowired
    private TestRestTemplate restTemplate;

    @Autowired
    private ElasticsearchService elasticsearchService;

    @Autowired
    private ObjectMapper objectMapper;

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("elasticsearch.host", elasticsearch::getHost);
        registry.add("elasticsearch.port", elasticsearch::getFirstMappedPort);
    }

    @BeforeEach
    void setUp() throws IOException {
        // Clear and recreate test data
        clearTestData();
        createTestData();
        
        // Wait for Elasticsearch to index the data
        await().atMost(10, SECONDS)
            .until(() -> countDocuments() > 0);
    }

    @AfterEach
    void tearDown() throws IOException {
        clearTestData();
    }

    @Test
    @WithMockUser(roles = "LOG_VIEWER")
    @DisplayName("Should search logs with username filter")
    void testSearchLogs_UsernameFilter() throws Exception {
        // Given
        String url = "/api/v1/logs/search?username=sirat&page=0&size=10";
        
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + generateTestJwtToken());
        HttpEntity<String> entity = new HttpEntity<>(headers);

        // When
        ResponseEntity<String> response = restTemplate.exchange(
            url, HttpMethod.GET, entity, String.class);

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        
        JsonNode responseBody = objectMapper.readTree(response.getBody());
        assertThat(responseBody.get("success").asBoolean()).isTrue();
        
        JsonNode data = responseBody.get("data");
        assertThat(data.get("totalElements").asLong()).isGreaterThan(0);
        
        JsonNode content = data.get("content");
        assertThat(content).isArray();
        content.forEach(logEntry -> {
            assertThat(logEntry.get("username").asText()).isEqualTo("sirat");
        });
    }

    @Test
    @WithMockUser(roles = "LOG_VIEWER")
    @DisplayName("Should search logs with date range filter")
    void testSearchLogs_DateRangeFilter() throws Exception {
        // Given
        LocalDateTime now = LocalDateTime.now();
        LocalDateTime yesterday = now.minusDays(1);
        
        String url = String.format(
            "/api/v1/logs/search?startTime=%s&endTime=%s",
            URLEncoder.encode(yesterday.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")), "UTF-8"),
            URLEncoder.encode(now.format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss")), "UTF-8")
        );
        
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + generateTestJwtToken());
        HttpEntity<String> entity = new HttpEntity<>(headers);

        // When
        ResponseEntity<String> response = restTemplate.exchange(
            url, HttpMethod.GET, entity, String.class);

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        
        JsonNode responseBody = objectMapper.readTree(response.getBody());
        assertThat(responseBody.get("success").asBoolean()).isTrue();
        
        JsonNode content = responseBody.get("data").get("content");
        content.forEach(logEntry -> {
            String timestamp = logEntry.get("timestamp").asText();
            LocalDateTime logTime = LocalDateTime.parse(timestamp, 
                DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSS"));
            assertThat(logTime).isBetween(yesterday, now);
        });
    }

    @Test
    @WithMockUser(roles = "LOG_VIEWER")
    @DisplayName("Should return paginated results correctly")
    void testSearchLogs_Pagination() throws Exception {
        // Given
        String url = "/api/v1/logs/search?page=0&size=5";
        
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + generateTestJwtToken());
        HttpEntity<String> entity = new HttpEntity<>(headers);

        // When
        ResponseEntity<String> response = restTemplate.exchange(
            url, HttpMethod.GET, entity, String.class);

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        
        JsonNode responseBody = objectMapper.readTree(response.getBody());
        JsonNode data = responseBody.get("data");
        
        assertThat(data.get("page").asInt()).isEqualTo(0);
        assertThat(data.get("size").asInt()).isEqualTo(5);
        assertThat(data.get("content").size()).isLessThanOrEqualTo(5);
        assertThat(data.get("totalElements").asLong()).isGreaterThan(0);
    }

    @Test
    @DisplayName("Should return 401 for unauthorized requests")
    void testSearchLogs_Unauthorized() {
        // Given
        String url = "/api/v1/logs/search?username=test";

        // When
        ResponseEntity<String> response = restTemplate.getForEntity(url, String.class);

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.UNAUTHORIZED);
    }

    @Test
    @WithMockUser(roles = "INVALID_ROLE")
    @DisplayName("Should return 403 for insufficient permissions")
    void testSearchLogs_Forbidden() {
        // Given
        String url = "/api/v1/logs/search";
        
        HttpHeaders headers = new HttpHeaders();
        headers.set("Authorization", "Bearer " + generateTestJwtToken("INVALID_ROLE"));
        HttpEntity<String> entity = new HttpEntity<>(headers);

        // When
        ResponseEntity<String> response = restTemplate.exchange(
            url, HttpMethod.GET, entity, String.class);

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.FORBIDDEN);
    }

    // Helper methods
    private void createTestData() throws IOException {
        String[] testLogEntries = {
            createTestLogJson("sirat", "EDCS", "LOGIN", "192.168.1.100"),
            createTestLogJson("sirat", "EDCS", "DOWNLOAD", "192.168.1.100"),
            createTestLogJson("somchai", "LEAV", "LOGIN", "10.1.1.50"),
            createTestLogJson("admin", "OFC-APP", "VIEW", "192.168.1.200"),
            createTestLogJson("sirat", "LEAV", "LOGOUT", "192.168.1.100")
        };
        
        for (int i = 0; i < testLogEntries.length; i++) {
            IndexRequest request = new IndexRequest("activity-logs-test")
                .id(String.valueOf(i + 1))
                .source(testLogEntries[i], XContentType.JSON);
            
            elasticsearchService.index(request);
        }
        
        // Refresh index to make documents searchable
        elasticsearchService.refresh("activity-logs-test");
    }
    
    private String createTestLogJson(String username, String system, String action, String ipAddress) {
        try {
            Map<String, Object> logEntry = Map.of(
                "timestamp", LocalDateTime.now().minusMinutes(new Random().nextInt(60))
                    .format(DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss.SSS")),
                "system", system,
                "username", username,
                "action", action,
                "ipAddress", ipAddress,
                "sessionId", "sess_" + UUID.randomUUID().toString().substring(0, 8),
                "status", "SUCCESS"
            );
            
            return objectMapper.writeValueAsString(logEntry);
        } catch (Exception e) {
            throw new RuntimeException("Failed to create test log JSON", e);
        }
    }
    
    private void clearTestData() throws IOException {
        try {
            DeleteByQueryRequest deleteRequest = new DeleteByQueryRequest("activity-logs-test");
            deleteRequest.setQuery(QueryBuilders.matchAllQuery());
            elasticsearchService.deleteByQuery(deleteRequest);
        } catch (Exception e) {
            // Index might not exist, ignore
        }
    }
    
    private long countDocuments() throws IOException {
        try {
            CountRequest countRequest = new CountRequest("activity-logs-test");
            countRequest.setQuery(QueryBuilders.matchAllQuery());
            CountResponse response = elasticsearchService.count(countRequest);
            return response.getCount();
        } catch (Exception e) {
            return 0;
        }
    }
    
    private String generateTestJwtToken(String... roles) {
        // Implementation depends on JWT library used
        return "test-jwt-token"; // Simplified for example
    }
}
```

### Database Integration Tests

#### Elasticsearch Integration Tests
```java
@Testcontainers
@TestPropertySource(properties = "spring.profiles.active=test")
@DisplayName("Elasticsearch Integration Tests")
class ElasticsearchIntegrationTest {

    @Container
    static ElasticsearchContainer elasticsearch = new ElasticsearchContainer(
        "docker.elastic.co/elasticsearch/elasticsearch:7.15.0")
        .withExposedPorts(9200)
        .withEnv("discovery.type", "single-node")
        .withEnv("xpack.security.enabled", "false");

    private ElasticsearchService elasticsearchService;
    private RestHighLevelClient client;

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("elasticsearch.host", elasticsearch::getHost);
        registry.add("elasticsearch.port", elasticsearch::getFirstMappedPort);
    }

    @BeforeEach
    void setUp() {
        // Initialize Elasticsearch client and service
        client = createElasticsearchClient();
        elasticsearchService = new ElasticsearchService(client);
    }

    @AfterEach
    void tearDown() throws IOException {
        if (client != null) {
            client.close();
        }
    }

    @Test
    @DisplayName("Should create index with proper mapping")
    void testCreateIndexWithMapping() throws IOException {
        // Given
        String indexName = "test-activity-logs";
        String mappingJson = createIndexMappingJson();

        // When
        CreateIndexRequest request = new CreateIndexRequest(indexName);
        request.mapping(mappingJson, XContentType.JSON);
        
        CreateIndexResponse response = client.indices().create(request, RequestOptions.DEFAULT);

        // Then
        assertThat(response.isAcknowledged()).isTrue();
        
        // Verify mapping was applied correctly
        GetMappingsRequest getMappingRequest = new GetMappingsRequest().indices(indexName);
        GetMappingsResponse mappingResponse = client.indices()
            .getMapping(getMappingRequest, RequestOptions.DEFAULT);
        
        assertThat(mappingResponse.mappings()).containsKey(indexName);
    }

    @Test
    @DisplayName("Should index and search documents correctly")
    void testIndexAndSearchDocuments() throws IOException {
        // Given
        String indexName = "test-logs";
        createTestIndex(indexName);
        
        LogEntry logEntry = LogEntry.builder()
            .timestamp(LocalDateTime.now())
            .system("EDCS")
            .username("testuser")
            .action("LOGIN")
            .ipAddress("192.168.1.100")
            .sessionId("test-session-123")
            .status("SUCCESS")
            .build();

        // When - Index document
        IndexRequest indexRequest = new IndexRequest(indexName)
            .id("test-1")
            .source(convertLogEntryToJson(logEntry), XContentType.JSON);
        
        IndexResponse indexResponse = client.index(indexRequest, RequestOptions.DEFAULT);
        
        // Refresh index to make document searchable
        RefreshRequest refreshRequest = new RefreshRequest(indexName);
        client.indices().refresh(refreshRequest, RequestOptions.DEFAULT);

        // When - Search document
        SearchRequest searchRequest = new SearchRequest(indexName);
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.query(QueryBuilders.termQuery("username", "testuser"));
        searchRequest.source(sourceBuilder);
        
        SearchResponse searchResponse = client.search(searchRequest, RequestOptions.DEFAULT);

        // Then
        assertThat(indexResponse.getResult()).isEqualTo(DocWriteResponse.Result.CREATED);
        assertThat(searchResponse.getHits().getTotalHits().value).isEqualTo(1L);
        
        SearchHit hit = searchResponse.getHits().getAt(0);
        assertThat(hit.getSourceAsMap().get("username")).isEqualTo("testuser");
        assertThat(hit.getSourceAsMap().get("system")).isEqualTo("EDCS");
    }

    @Test
    @DisplayName("Should perform complex aggregation queries")
    void testComplexAggregationQueries() throws IOException {
        // Given
        String indexName = "test-aggregation-logs";
        createTestIndex(indexName);
        indexMultipleTestDocuments(indexName);

        // When - Execute aggregation query
        SearchRequest searchRequest = new SearchRequest(indexName);
        SearchSourceBuilder sourceBuilder = new SearchSourceBuilder();
        sourceBuilder.size(0); // Only aggregations, no documents
        
        // Group by system and action
        TermsAggregationBuilder systemAgg = AggregationBuilders
            .terms("by_system")
            .field("system.keyword")
            .subAggregation(
                AggregationBuilders.terms("by_action").field("action.keyword")
            );
        
        sourceBuilder.aggregation(systemAgg);
        searchRequest.source(sourceBuilder);
        
        SearchResponse response = client.search(searchRequest, RequestOptions.DEFAULT);

        // Then
        assertThat(response.getAggregations()).isNotNull();
        
        Terms systemTerms = response.getAggregations().get("by_system");
        assertThat(systemTerms.getBuckets()).isNotEmpty();
        
        for (Terms.Bucket systemBucket : systemTerms.getBuckets()) {
            Terms actionTerms = systemBucket.getAggregations().get("by_action");
            assertThat(actionTerms.getBuckets()).isNotEmpty();
        }
    }

    @Test
    @DisplayName("Should handle bulk operations efficiently")
    void testBulkOperations() throws IOException {
        // Given
        String indexName = "test-bulk-logs";
        createTestIndex(indexName);
        
        List<LogEntry> logEntries = createBulkTestData(1000);

        // When - Bulk index operation
        BulkRequest bulkRequest = new BulkRequest();
        
        for (int i = 0; i < logEntries.size(); i++) {
            IndexRequest indexRequest = new IndexRequest(indexName)
                .id(String.valueOf(i))
                .source(convertLogEntryToJson(logEntries.get(i)), XContentType.JSON);
            bulkRequest.add(indexRequest);
        }
        
        BulkResponse bulkResponse = client.bulk(bulkRequest, RequestOptions.DEFAULT);

        // Then
        assertThat(bulkResponse.hasFailures()).isFalse();
        assertThat(bulkResponse.getItems()).hasSize(1000);
        
        // Verify documents were indexed
        RefreshRequest refreshRequest = new RefreshRequest(indexName);
        client.indices().refresh(refreshRequest, RequestOptions.DEFAULT);
        
        CountRequest countRequest = new CountRequest(indexName);
        CountResponse countResponse = client.count(countRequest, RequestOptions.DEFAULT);
        assertThat(countResponse.getCount()).isEqualTo(1000L);
    }

    // Helper methods
    private RestHighLevelClient createElasticsearchClient() {
        HttpHost httpHost = new HttpHost(
            elasticsearch.getHost(),
            elasticsearch.getFirstMappedPort(),
            "http"
        );
        
        return new RestHighLevelClient(
            RestClient.builder(httpHost)
        );
    }
    
    private void createTestIndex(String indexName) throws IOException {
        CreateIndexRequest request = new CreateIndexRequest(indexName);
        request.mapping(createIndexMappingJson(), XContentType.JSON);
        client.indices().create(request, RequestOptions.DEFAULT);
    }
    
    private String createIndexMappingJson() {
        return """
        {
          "properties": {
            "timestamp": {"type": "date", "format": "yyyy-MM-dd HH:mm:ss.SSS"},
            "system": {"type": "keyword"},
            "username": {"type": "keyword"},
            "action": {"type": "keyword"},
            "ipAddress": {"type": "ip"},
            "sessionId": {"type": "keyword"},
            "status": {"type": "keyword"},
            "details": {"type": "text"},
            "responseTime": {"type": "long"}
          }
        }
        """;
    }
}
```

---

## 🚀 End-to-End Testing

### Selenium Web Tests

#### Dashboard E2E Tests
```java
@SpringBootTest(webEnvironment = SpringBootTest.WebEnvironment.DEFINED_PORT)
@Testcontainers
@TestPropertySource(properties = {
    "server.port=8080",
    "spring.profiles.active=test"
})
@DisplayName("Dashboard End-to-End Tests")
class DashboardE2ETest {

    @Container
    static ElasticsearchContainer elasticsearch = new ElasticsearchContainer(
        "docker.elastic.co/elasticsearch/elasticsearch:7.15.0")
        .withExposedPorts(9200)
        .withEnv("discovery.type", "single-node")
        .withEnv("xpack.security.enabled", "false");

    private WebDriver driver;
    private WebDriverWait wait;

    @DynamicPropertySource
    static void configureProperties(DynamicPropertyRegistry registry) {
        registry.add("elasticsearch.host", elasticsearch::getHost);
        registry.add("elasticsearch.port", elasticsearch::getFirstMappedPort);
    }

    @BeforeEach
    void setUp() {
        ChromeOptions options = new ChromeOptions();
        options.addArguments("--headless", "--no-sandbox", "--disable-dev-shm-usage");
        driver = new ChromeDriver(options);
        wait = new WebDriverWait(driver, Duration.ofSeconds(10));
        
        // Setup test data in Elasticsearch
        setupTestData();
    }

    @AfterEach
    void tearDown() {
        if (driver != null) {
            driver.quit();
        }
    }

    @Test
    @DisplayName("Should display dashboard with correct statistics")
    void testDashboardDisplay() {
        // Given
        driver.get("http://localhost:8080/activity-tracking/dashboard");
        
        // Login if required
        performLogin("test_user", "test_password");

        // When - Wait for dashboard to load
        wait.until(ExpectedConditions.presenceOfElementLocated(
            By.id("dashboard-container")));

        // Then - Verify dashboard components
        WebElement totalUsersCard = driver.findElement(By.id("total-users-card"));
        assertThat(totalUsersCard.isDisplayed()).isTrue();
        
        WebElement totalSessionsCard = driver.findElement(By.id("total-sessions-card"));
        assertThat(totalSessionsCard.isDisplayed()).isTrue();
        
        WebElement authStatsChart = driver.findElement(By.id("auth-stats-chart"));
        assertThat(authStatsChart.isDisplayed()).isTrue();
        
        // Verify data is loaded
        wait.until(ExpectedConditions.textToBePresentInElement(
            totalUsersCard, "Total Users"));
        
        String usersCount = driver.findElement(By.id("users-count")).getText();
        assertThat(Integer.parseInt(usersCount)).isGreaterThan(0);
    }

    @Test
    @DisplayName("Should filter data based on time range selection")
    void testTimeRangeFilter() {
        // Given
        driver.get("http://localhost:8080/activity-tracking/dashboard");
        performLogin("test_user", "test_password");

        // When - Select different time range
        WebElement timeRangeDropdown = wait.until(
            ExpectedConditions.elementToBeClickable(By.id("time-range-selector")));
        timeRangeDropdown.click();
        
        WebElement last7DaysOption = driver.findElement(By.xpath("//option[@value='7d']"));
        last7DaysOption.click();
        
        // Wait for data to refresh
        wait.until(ExpectedConditions.attributeContains(
            By.id("dashboard-container"), "data-loading", "false"));

        // Then - Verify data updated
        WebElement timeRangeDisplay = driver.findElement(By.id("current-time-range"));
        assertThat(timeRangeDisplay.getText()).contains("Last 7 days");
        
        // Verify charts updated with new data
        List<WebElement> chartElements = driver.findElements(By.className("chart-container"));
        assertThat(chartElements).allMatch(WebElement::isDisplayed);
    }

    @Test
    @DisplayName("Should navigate to log search from dashboard")
    void testNavigationToLogSearch() {
        // Given
        driver.get("http://localhost:8080/activity-tracking/dashboard");
        performLogin("test_user", "test_password");

        // When - Click on log search navigation
        WebElement logSearchLink = wait.until(
            ExpectedConditions.elementToBeClickable(By.id("nav-log-search")));
        logSearchLink.click();

        // Then - Verify navigation to log search page
        wait.until(ExpectedConditions.urlContains("/log-search"));
        
        WebElement searchForm = driver.findElement(By.id("log-search-form"));
        assertThat(searchForm.isDisplayed()).isTrue();
        
        WebElement usernameField = driver.findElement(By.id("username-filter"));
        assertThat(usernameField.isDisplayed()).isTrue();
    }

    @Test
    @DisplayName("Should perform log search and display results")
    void testLogSearchFunctionality() {
        // Given
        driver.get("http://localhost:8080/activity-tracking/log-search");
        performLogin("test_user", "test_password");

        // When - Enter search criteria
        WebElement usernameField = wait.until(
            ExpectedConditions.presenceOfElementLocated(By.id("username-filter")));
        usernameField.sendKeys("sirat");
        
        WebElement systemDropdown = driver.findElement(By.id("system-filter"));
        Select systemSelect = new Select(systemDropdown);
        systemSelect.selectByValue("EDCS");
        
        WebElement searchButton = driver.findElement(By.id("search-button"));
        searchButton.click();

        // Then - Verify search results
        wait.until(ExpectedConditions.presenceOfElementLocated(By.id("search-results")));
        
        List<WebElement> resultRows = driver.findElements(By.className("log-entry-row"));
        assertThat(resultRows).isNotEmpty();
        
        // Verify result data matches search criteria
        WebElement firstRow = resultRows.get(0);
        String username = firstRow.findElement(By.className("username-cell")).getText();
        String system = firstRow.findElement(By.className("system-cell")).getText();
        
        assertThat(username).isEqualTo("sirat");
        assertThat(system).isEqualTo("EDCS");
    }

    @Test
    @DisplayName("Should handle error states gracefully")
    void testErrorHandling() {
        // Given - Stop Elasticsearch to simulate error
        elasticsearch.stop();
        
        driver.get("http://localhost:8080/activity-tracking/dashboard");
        performLogin("test_user", "test_password");

        // When - Dashboard tries to load data
        wait.until(ExpectedConditions.presenceOfElementLocated(By.id("dashboard-container")));

        // Then - Verify error message is displayed
        WebElement errorMessage = wait.until(
            ExpectedConditions.presenceOfElementLocated(By.className("error-message")));
        
        assertThat(errorMessage.isDisplayed()).isTrue();
        assertThat(errorMessage.getText()).contains("Unable to load dashboard data");
        
        // Verify retry button is available
        WebElement retryButton = driver.findElement(By.id("retry-button"));
        assertThat(retryButton.isDisplayed()).isTrue();
    }

    // Helper methods
    private void performLogin(String username, String password) {
        try {
            WebElement loginForm = driver.findElement(By.id("login-form"));
            
            WebElement usernameField = loginForm.findElement(By.name("username"));
            WebElement passwordField = loginForm.findElement(By.name("password"));
            WebElement loginButton = loginForm.findElement(By.type("submit"));
            
            usernameField.sendKeys(username);
            passwordField.sendKeys(password);
            loginButton.click();
            
            // Wait for login to complete
            wait.until(ExpectedConditions.urlContains("/dashboard"));
        } catch (NoSuchElementException e) {
            // Already logged in or no login required
        }
    }
    
    private void setupTestData() {
        // Implementation to setup test data in Elasticsearch
        // This would create sample log entries for testing
    }
}
```

---

## 📊 Performance Testing

### Load Testing with JMeter Scripts

#### API Load Test Configuration
```java
@DisplayName("Performance Tests")
class PerformanceTest {

    @Container
    static ElasticsearchContainer elasticsearch = new ElasticsearchContainer(
        "docker.elastic.co/elasticsearch/elasticsearch:7.15.0")
        .withExposedPorts(9200)
        .withEnv("discovery.type", "single-node")
        .withEnv("xpack.security.enabled", "false")
        .withEnv("ES_JAVA_OPTS", "-Xms1g -Xmx1g"); // Increased memory for performance testing

    private TestRestTemplate restTemplate;
    private ExecutorService executorService;

    @BeforeEach
    void setUp() {
        restTemplate = new TestRestTemplate();
        executorService = Executors.newFixedThreadPool(50);
        
        // Create large dataset for performance testing
        createLargeTestDataset();
    }

    @AfterEach
    void tearDown() {
        executorService.shutdown();
    }

    @Test
    @DisplayName("Should handle concurrent log search requests")
    void testConcurrentLogSearchPerformance() throws InterruptedException {
        // Given
        int numberOfConcurrentUsers = 50;
        int requestsPerUser = 10;
        CountDownLatch latch = new CountDownLatch(numberOfConcurrentUsers);
        List<Future<ResponseTimeResult>> futures = new ArrayList<>();

        // When - Execute concurrent requests
        for (int i = 0; i < numberOfConcurrentUsers; i++) {
            Future<ResponseTimeResult> future = executorService.submit(() -> {
                try {
                    List<Long> responseTimes = new ArrayList<>();
                    
                    for (int j = 0; j < requestsPerUser; j++) {
                        long startTime = System.currentTimeMillis();
                        
                        ResponseEntity<String> response = restTemplate.getForEntity(
                            "http://localhost:8080/api/v1/logs/search?username=sirat&page=0&size=20",
                            String.class
                        );
                        
                        long endTime = System.currentTimeMillis();
                        responseTimes.add(endTime - startTime);
                        
                        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
                    }
                    
                    return new ResponseTimeResult(responseTimes);
                } finally {
                    latch.countDown();
                }
            });
            
            futures.add(future);
        }

        // Wait for all requests to complete
        latch.await(60, TimeUnit.SECONDS);

        // Then - Analyze performance results
        List<ResponseTimeResult> results = futures.stream()
            .map(future -> {
                try {
                    return future.get();
                } catch (Exception e) {
                    throw new RuntimeException(e);
                }
            })
            .collect(Collectors.toList());

        // Calculate statistics
        List<Long> allResponseTimes = results.stream()
            .flatMap(result -> result.getResponseTimes().stream())
            .collect(Collectors.toList());

        double averageResponseTime = allResponseTimes.stream()
            .mapToLong(Long::longValue)
            .average()
            .orElse(0.0);

        long maxResponseTime = allResponseTimes.stream()
            .mapToLong(Long::longValue)
            .max()
            .orElse(0L);

        long percentile95 = calculatePercentile(allResponseTimes, 95);

        // Performance assertions
        assertThat(averageResponseTime).isLessThan(2000.0); // Less than 2 seconds average
        assertThat(percentile95).isLessThan(5000L); // 95th percentile less than 5 seconds
        assertThat(maxResponseTime).isLessThan(10000L); // Maximum less than 10 seconds

        System.out.printf("Performance Results:%n");
        System.out.printf("Total Requests: %d%n", allResponseTimes.size());
        System.out.printf("Average Response Time: %.2f ms%n", averageResponseTime);
        System.out.printf("95th Percentile: %d ms%n", percentile95);
        System.out.printf("Max Response Time: %d ms%n", maxResponseTime);
    }

    @Test
    @DisplayName("Should handle large result sets efficiently")
    void testLargeResultSetPerformance() {
        // Given
        String url = "http://localhost:8080/api/v1/logs/search?size=1000"; // Large page size

        // When
        long startTime = System.currentTimeMillis();
        ResponseEntity<String> response = restTemplate.getForEntity(url, String.class);
        long endTime = System.currentTimeMillis();

        // Then
        assertThat(response.getStatusCode()).isEqualTo(HttpStatus.OK);
        assertThat(endTime - startTime).isLessThan(5000L); // Less than 5 seconds
    }

    // Helper classes and methods
    @Data
    @AllArgsConstructor
    private static class ResponseTimeResult {
        private List<Long> responseTimes;
    }

    private long calculatePercentile(List<Long> values, int percentile) {
        Collections.sort(values);
        int index = (int) Math.ceil((percentile / 100.0) * values.size()) - 1;
        return values.get(Math.max(0, index));
    }

    private void createLargeTestDataset() {
        // Implementation to create large dataset for performance testing
        // This would insert thousands of test log entries
    }
}
```

---

## 📋 Test Data Management

### Test Data Builders

#### Log Entry Test Data Builder
```java
@Component
public class LogEntryTestDataBuilder {

    private String username = "testuser";
    private String system = "EDCS";
    private String action = "LOGIN";
    private LocalDateTime timestamp = LocalDateTime.now();
    private String ipAddress = "192.168.1.100";
    private String sessionId = "test-session-123";
    private String status = "SUCCESS";

    public static LogEntryTestDataBuilder aLogEntry() {
        return new LogEntryTestDataBuilder();
    }

    public LogEntryTestDataBuilder withUsername(String username) {
        this.username = username;
        return this;
    }

    public LogEntryTestDataBuilder withSystem(String system) {
        this.system = system;
        return this;
    }

    public LogEntryTestDataBuilder withAction(String action) {
        this.action = action;
        return this;
    }

    public LogEntryTestDataBuilder withTimestamp(LocalDateTime timestamp) {
        this.timestamp = timestamp;
        return this;
    }

    public LogEntryTestDataBuilder withIpAddress(String ipAddress) {
        this.ipAddress = ipAddress;
        return this;
    }

    public LogEntryTestDataBuilder withSessionId(String sessionId) {
        this.sessionId = sessionId;
        return this;
    }

    public LogEntryTestDataBuilder withStatus(String status) {
        this.status = status;
        return this;
    }

    public LogEntryTestDataBuilder edcsLogin() {
        return withSystem("EDCS").withAction("LOGIN");
    }

    public LogEntryTestDataBuilder leavDownload() {
        return withSystem("LEAV").withAction("DOWNLOAD");
    }

    public LogEntryTestDataBuilder ofcAppView() {
        return withSystem("OFC-APP").withAction("VIEW");
    }

    public LogEntryTestDataBuilder failed() {
        return withStatus("FAILED");
    }

    public LogEntryTestDataBuilder fromLastWeek() {
        return withTimestamp(LocalDateTime.now().minusDays(7));
    }

    public LogEntry build() {
        return LogEntry.builder()
            .username(username)
            .system(system)
            .action(action)
            .timestamp(timestamp)
            .ipAddress(ipAddress)
            .sessionId(sessionId)
            .status(status)
            .build();
    }

    public List<LogEntry> buildList(int count) {
        List<LogEntry> entries = new ArrayList<>();
        for (int i = 0; i < count; i++) {
            entries.add(this.withSessionId("session-" + i).build());
        }
        return entries;
    }
}
```

### Test Data Factory
```java
@Component
public class TestDataFactory {

    public List<LogEntry> createRealisticTestData(int count) {
        List<LogEntry> entries = new ArrayList<>();
        Random random = new Random();
        
        String[] usernames = {"sirat", "somchai", "admin", "user1", "user2"};
        String[] systems = {"EDCS", "LEAV", "OFC-APP"};
        String[] actions = {"LOGIN", "LOGOUT", "DOWNLOAD", "UPLOAD", "VIEW", "MODIFY"};
        String[] ipAddresses = {"192.168.1.100", "192.168.1.101", "10.1.1.50", "10.1.1.51"};
        
        for (int i = 0; i < count; i++) {
            LocalDateTime timestamp = LocalDateTime.now()
                .minusMinutes(random.nextInt(10080)); // Random time within last week
            
            entries.add(LogEntry.builder()
                .username(usernames[random.nextInt(usernames.length)])
                .system(systems[random.nextInt(systems.length)])
                .action(actions[random.nextInt(actions.length)])
                .timestamp(timestamp)
                .ipAddress(ipAddresses[random.nextInt(ipAddresses.length)])
                .sessionId("sess_" + UUID.randomUUID().toString().substring(0, 8))
                .status(random.nextInt(10) == 0 ? "FAILED" : "SUCCESS") // 10% failure rate
                .documentId(random.nextBoolean() ? "DOC_" + random.nextInt(1000) : null)
                .responseTime(Duration.ofMillis(50 + random.nextInt(2000))) // 50ms to 2s
                .build());
        }
        
        return entries;
    }
}
```

---

## 📈 Test Reporting and Coverage

### JaCoCo Configuration
```xml
<plugin>
    <groupId>org.jacoco</groupId>
    <artifactId>jacoco-maven-plugin</artifactId>
    <version>0.8.8</version>
    <executions>
        <execution>
            <goals>
                <goal>prepare-agent</goal>
            </goals>
        </execution>
        <execution>
            <id>report</id>
            <phase>test</phase>
            <goals>
                <goal>report</goal>
            </goals>
        </execution>
        <execution>
            <id>jacoco-check</id>
            <goals>
                <goal>check</goal>
            </goals>
            <configuration>
                <rules>
                    <rule>
                        <element>PACKAGE</element>
                        <limits>
                            <limit>
                                <counter>LINE</counter>
                                <value>COVEREDRATIO</value>
                                <minimum>0.80</minimum>
                            </limit>
                        </limits>
                    </rule>
                </rules>
            </configuration>
        </execution>
    </executions>
</plugin>
```

### Test Execution Commands
```bash
# Run all tests with coverage
mvn clean test jacoco:report

# Run only unit tests
mvn test -Dtest=**/*Test

# Run only integration tests
mvn test -Dtest=**/*IntegrationTest

# Run performance tests
mvn test -Dtest=**/*PerformanceTest -Dspring.profiles.active=performance

# Generate test report
mvn surefire-report:report

# Run tests with specific profile
mvn test -Dspring.profiles.active=test -Delasticsearch.host=localhost

# Skip tests (for quick builds)
mvn install -DskipTests

# Run tests in parallel
mvn test -Dparallel=all -DthreadCount=4
```

---

**Next:** [Deployment Guide](./Deployment-Guide.md)  
**See Also:** [Developer Setup Guide](./Developer-Setup-Guide.md) | [API Development Guide](./API-Development-Guide.md)
