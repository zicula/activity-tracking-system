# Design Document - ACTIVITY-TRACKING System

## Document Information
**Project Name:** ACTIVITY-TRACKING  
**Version:** 1.0  
**Date:** July 23, 2025  
**Based on:** Requirements Document v1.0  
**Author:** Development Team  
**Reviewers:** [Pending]

## Executive Summary
ACTIVITY-TRACKING เป็นระบบติดตามพฤติกรรมของผู้ใช้งานในระบบ E-Office อย่างครอบคลุมและละเอียด ออกแบบมาเพื่อเก็บบันทึกทุกการปฏิสัมพันธ์ของผู้ใช้งาน รวมถึง:

**Comprehensive User Behavior Tracking:**
- **Click-level tracking**: ทุกการกดปุ่ม, การคลิกลิงก์, การเลือกเมนู
- **User interaction patterns**: Mouse movements, scroll behavior, hover events
- **Session flow tracking**: การเนวิเกตระหว่างหน้า, time spent per screen
- **Form interaction tracking**: การกรอกข้อมูล, การเปลี่ยนแปลงฟิลด์
- **Document interaction tracking**: การเปิด, อ่าน, download, แชร์เอกสาร
- **Approval workflow tracking**: ทุกขั้นตอนการอนุมัติ, การตัดสินใจ
- **Error and exception tracking**: การเกิด error, การใช้งานที่ไม่สำเร็จ

ระบบออกแบบให้รองรับผู้ใช้งาน 2,000-8,000 คน และสอดคล้องกับข้อกำหนด PDPA โดยเก็บข้อมูลพฤติกรรมแบบ anonymized เพื่อการวิเคราะห์และปรับปรุงประสบการณ์ผู้ใช้งาน

---

## System Architecture

### High-Level Architecture
```
┌─────────────────────────────────────────────────────────────────────┐
│                    E-OFFICE ECOSYSTEM                              │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐ ┌─────────────┐   │
│  │   Portal    │ │ Doc Mgmt    │ │ Approval    │ │ HR Portal   │   │
│  │             │ │             │ │  System     │ │             │   │
│  └─────────────┘ └─────────────┘ └─────────────┘ └─────────────┘   │
│         │               │               │               │           │
│         └───────────────┼───────────────┼───────────────┘           │
│                         │               │                           │
└─────────────────────────┼───────────────┼───────────────────────────┘
                          │               │
                          ▼               ▼
┌─────────────────────────────────────────────────────────────────────┐
│                ACTIVITY-TRACKING SYSTEM                            │
├─────────────────────────────────────────────────────────────────────┤
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                   API GATEWAY                               │   │
│  │         (Authentication & Rate Limiting)                    │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                │                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                TRACKING SERVICES                            │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │   │
│  │  │  User Auth  │ │ Doc Access  │ │  Approval   │          │   │
│  │  │  Tracker    │ │  Tracker    │ │  Tracker    │          │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘          │   │
│  │  ┌─────────────┐ ┌─────────────┐                          │   │
│  │  │   Module    │ │ Real-time   │                          │   │
│  │  │  Tracker    │ │ Processor   │                          │   │
│  │  └─────────────┘ └─────────────┘                          │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                │                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │                 DATA LAYER                                  │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │   │
│  │  │   Oracle    │ │    Redis    │ │ Kafka/MQ    │          │   │
│  │  │  Database   │ │   Cache     │ │   Stream    │          │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘          │   │
│  └─────────────────────────────────────────────────────────────┘   │
│                                │                                    │
│  ┌─────────────────────────────────────────────────────────────┐   │
│  │              PRESENTATION LAYER                             │   │
│  │  ┌─────────────┐ ┌─────────────┐ ┌─────────────┐          │   │
│  │  │ Real-time   │ │ Analytics   │ │   Report    │          │   │
│  │  │ Dashboard   │ │  Portal     │ │ Generator   │          │   │
│  │  └─────────────┘ └─────────────┘ └─────────────┘          │   │
│  └─────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────┘
```

### Component Architecture
| Component | Responsibility | Technology | Scale |
|-----------|---------------|------------|--------|
| API Gateway | Request routing, auth, rate limiting | Nginx + Java | 4,000 concurrent |
| Behavior Tracking Engine | Click, scroll, mouse movement tracking | Java Spring Boot | Very High throughput |
| Session Flow Tracker | Page navigation, user journey mapping | Java Spring Boot | High throughput |
| Interaction Collector | Form inputs, button clicks, UI events | Java Spring Boot | Very High throughput |
| Document Behavior Tracker | Document viewing patterns, interactions | Java Spring Boot | Medium throughput |
| Approval Flow Tracker | Workflow step tracking, decision patterns | Java Spring Boot | Medium throughput |
| Real-time Processor | Event stream processing & aggregation | Java + Kafka | Real-time |
| Behavior Analytics Engine | Pattern recognition, user journey analysis | Java + ML libraries | Batch processing |
| Oracle Database | Primary data storage | Oracle 19c+ | 800GB/year |
| Redis Cache | Session & real-time behavior data | Redis Cluster | Memory-optimized |
| Dashboard Frontend | Behavior analytics & visualization | React/Angular | Responsive |

---

## Technical Specifications

### Technology Stack

#### Backend Technologies
**Primary Framework:** Java Spring Boot 2.7+
- **Rationale:** Enterprise-grade, well-supported, team expertise
- **Migration Path:** Java 1.8 → Java 18 during development cycle

**API Layer:** Spring Boot + Spring Security
- **Authentication:** JWT-based with refresh tokens
- **Authorization:** Role-based access control (RBAC)
- **Rate Limiting:** Redis-based sliding window

**Microservices Architecture:**
- **Service Discovery:** Eureka Server
- **Configuration Management:** Spring Cloud Config
- **Circuit Breaker:** Hystrix/Resilience4j

#### Database Technologies
**Primary Database:** Oracle Database 19c+
- **Connection Pool:** HikariCP (high-performance)
- **ORM:** JPA/Hibernate with Oracle dialect
- **Migration:** Liquibase for schema versioning

**Caching Layer:** Redis Cluster
- **Session Storage:** User sessions and temporary data
- **Real-time Cache:** Dashboard metrics and live data
- **Cache Strategy:** Write-through with TTL

**Message Queue:** Apache Kafka
- **Event Streaming:** Real-time activity events
- **Topic Structure:** Partitioned by activity type
- **Retention:** 7 days for high-throughput topics

#### Frontend Technologies
**JavaScript Tracking SDK:** Comprehensive client-side behavior tracking
- **Event Capture:** Click, scroll, mouse movement, keyboard events
- **User Journey Mapping:** Page navigation, session flow tracking
- **Form Interaction Tracking:** Field changes, validation errors, submission patterns
- **Performance Monitoring:** Page load times, AJAX response times
- **Error Tracking:** JavaScript errors, failed API calls
- **Viewport Tracking:** Screen dimensions, scroll position, element visibility

**Dashboard Framework:** React 18+ with TypeScript
- **State Management:** Redux Toolkit
- **UI Components:** Ant Design or Material-UI
- **Charts:** D3.js for complex behavior visualizations
- **Heat Maps:** User interaction heat maps and click density
- **Flow Diagrams:** User journey and conversion funnel visualization

**Real-time Updates:** WebSocket + Socket.IO
- **Live Behavior Streaming:** Real-time user activity monitoring
- **Session Replay:** Capability to replay user sessions (privacy-compliant)
- **Connection Management:** Automatic reconnection
- **Event Broadcasting:** Room-based subscriptions

#### Infrastructure Technologies
**Containerization:** Docker + Docker Compose
- **Base Images:** OpenJDK 18 Alpine
- **Multi-stage builds:** Optimized image sizes
- **Health Checks:** Built-in container health monitoring

**Load Balancing:** Nginx
- **SSL Termination:** TLS 1.3 support
- **Compression:** Gzip/Brotli compression
- **Static Assets:** CDN-ready configuration

---

## Database Design

### Entity Relationship Diagram
```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│     USERS       │     │    SESSIONS     │     │ USER_BEHAVIORS  │
├─────────────────┤     ├─────────────────┤     ├─────────────────┤
│ user_id (PK)    │────▶│ session_id (PK) │────▶│ behavior_id(PK) │
│ user_hash       │     │ user_id (FK)    │     │ session_id (FK) │
│ department_id   │     │ login_time      │     │ event_type      │
│ role_level      │     │ logout_time     │     │ timestamp       │
│ created_date    │     │ ip_address_hash │     │ element_id      │
│ is_active       │     │ user_agent_hash │     │ element_type    │
└─────────────────┘     │ session_status  │     │ page_url        │
                        │ viewport_size   │     │ coordinates_x   │
                        │ device_type     │     │ coordinates_y   │
                        └─────────────────┘     │ scroll_position │
                                                │ duration_ms     │
                                                │ metadata_json   │
                                                └─────────────────┘

┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  PAGE_VIEWS     │     │ FORM_INTERACTIONS│     │  CLICK_EVENTS   │
├─────────────────┤     ├─────────────────┤     ├─────────────────┤
│ page_view_id(PK)│     │ form_interact_id│     │ click_event_id  │
│ session_id (FK) │     │ session_id (FK) │     │ session_id (FK) │
│ page_url        │     │ form_id         │     │ element_id      │
│ page_title      │     │ field_name      │     │ element_class   │
│ entry_time      │     │ action_type     │     │ element_text    │
│ exit_time       │     │ field_value_hash│     │ click_timestamp │
│ time_on_page    │     │ timestamp       │     │ coordinates_x   │
│ scroll_depth    │     │ validation_error│     │ coordinates_y   │
│ referrer_url    │     └─────────────────┘     │ button_type     │
└─────────────────┘                             └─────────────────┘

┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  MOUSE_TRACKING │     │ SCROLL_BEHAVIORS│     │   ERROR_EVENTS  │
├─────────────────┤     ├─────────────────┤     ├─────────────────┤
│ mouse_track_id  │     │ scroll_id (PK)  │     │ error_id (PK)   │
│ session_id (FK) │     │ session_id (FK) │     │ session_id (FK) │
│ page_url        │     │ page_url        │     │ error_type      │
│ movement_path   │     │ scroll_start    │     │ error_message   │
│ hover_elements  │     │ scroll_end      │     │ stack_trace     │
│ timestamp       │     │ scroll_speed    │     │ page_url        │
│ coordinates_x   │     │ direction       │     │ timestamp       │
│ coordinates_y   │     │ timestamp       │     │ user_agent      │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

### Core Tables

#### USERS Table
```sql
CREATE TABLE users (
    user_id         NUMBER(10) PRIMARY KEY,
    user_hash       VARCHAR2(64) NOT NULL UNIQUE,  -- SHA-256 hash
    department_id   NUMBER(5),
    role_level      NUMBER(2),  -- 1=User, 2=Manager, 3=Admin
    created_date    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_active     TIMESTAMP,
    is_active       NUMBER(1) DEFAULT 1,
    CONSTRAINT ck_role_level CHECK (role_level IN (1,2,3))
);
```

#### SESSIONS Table
```sql
CREATE TABLE sessions (
    session_id      VARCHAR2(128) PRIMARY KEY,
    user_id         NUMBER(10) NOT NULL,
    login_time      TIMESTAMP NOT NULL,
    logout_time     TIMESTAMP,
    ip_address_hash VARCHAR2(64),  -- Masked for privacy
    user_agent_hash VARCHAR2(64),  -- Hashed user agent
    session_status  VARCHAR2(20) DEFAULT 'ACTIVE',
    created_date    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(user_id),
    CONSTRAINT ck_session_status CHECK (session_status IN ('ACTIVE','EXPIRED','LOGOUT'))
);
```

#### ACTIVITIES Table
```sql
CREATE TABLE activities (
    activity_id     NUMBER(15) PRIMARY KEY,
    session_id      VARCHAR2(128) NOT NULL,
    activity_type   VARCHAR2(50) NOT NULL,
    timestamp       TIMESTAMP NOT NULL,
    module_name     VARCHAR2(100),
    action_details  VARCHAR2(500),
    duration_ms     NUMBER(10),
    metadata_json   CLOB,  -- Flexible metadata storage
    created_date    TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (session_id) REFERENCES sessions(session_id),
    CONSTRAINT ck_activity_type CHECK (activity_type IN (
        'LOGIN','LOGOUT','PAGE_VIEW','DOCUMENT_ACCESS','APPROVAL_ACTION','MODULE_ACCESS'
    ))
);
```

### Indexing Strategy
```sql
-- Performance indexes
CREATE INDEX idx_activities_timestamp ON activities(timestamp);
CREATE INDEX idx_activities_type_timestamp ON activities(activity_type, timestamp);
CREATE INDEX idx_sessions_user_login ON sessions(user_id, login_time);
CREATE INDEX idx_activities_session ON activities(session_id);

-- Partitioning strategy (by month)
ALTER TABLE activities PARTITION BY RANGE (timestamp) (
    PARTITION p_202507 VALUES LESS THAN (DATE '2025-08-01'),
    PARTITION p_202508 VALUES LESS THAN (DATE '2025-09-01'),
    -- Add partitions monthly via automated script
);
```

---

## API Design

### RESTful API Endpoints

#### Authentication APIs
```
POST /api/v1/auth/track-login
POST /api/v1/auth/track-logout
GET  /api/v1/auth/session/{sessionId}
```

#### Activity Tracking APIs
```
POST /api/v1/activities/track
GET  /api/v1/activities/user/{userId}
GET  /api/v1/activities/session/{sessionId}
POST /api/v1/activities/batch
```

#### Dashboard APIs
```
GET  /api/v1/dashboard/realtime
GET  /api/v1/dashboard/metrics
GET  /api/v1/dashboard/alerts
```

#### Reporting APIs
```
GET  /api/v1/reports/user-activity
GET  /api/v1/reports/department-summary
POST /api/v1/reports/custom
GET  /api/v1/reports/export/{reportId}
```

### API Specifications

#### 1. Behavior Tracking Endpoints
```json
POST /api/v1/behaviors/events/bulk
Headers: {
  "Authorization": "Bearer {jwt_token}",
  "Content-Type": "application/json"
}

Request Body:
{
  "sessionId": "sess_123456789",
  "events": [
    {
      "eventType": "CLICK",
      "timestamp": "2025-07-23T10:30:00.123Z",
      "elementId": "btn-submit",
      "elementType": "button",
      "coordinates": {"x": 450, "y": 320},
      "pageUrl": "/documents/approval",
      "metadata": {
        "elementText": "Submit Approval",
        "elementClass": "btn-primary",
        "formId": "approval-form-001"
      }
    },
    {
      "eventType": "SCROLL",
      "timestamp": "2025-07-23T10:30:02.456Z",
      "scrollPosition": 850,
      "direction": "down",
      "speed": 120,
      "pageUrl": "/documents/approval"
    }
  ]
}

Response:
{
  "status": "success",
  "processedEvents": 2,
  "eventIds": ["evt_001", "evt_002"],
  "timestamp": "2025-07-23T10:30:02.500Z"
}
```

#### 2. Mouse Movement Tracking
```json
POST /api/v1/behaviors/mouse/track
Headers: {
  "Authorization": "Bearer {jwt_token}",
  "Content-Type": "application/json"
}

Request Body:
{
  "sessionId": "sess_123456789",
  "pageUrl": "/documents/review",
  "movementData": {
    "path": [
      {"x": 100, "y": 200, "timestamp": 1625123456789},
      {"x": 150, "y": 220, "timestamp": 1625123456820},
      {"x": 200, "y": 240, "timestamp": 1625123456850}
    ],
    "hoverElements": [
      {
        "elementId": "doc-title",
        "hoverDuration": 1200,
        "timestamp": "2025-07-23T10:30:01.000Z"
      }
    ]
  }
}
```

#### 3. Form Interaction Analytics
```json
POST /api/v1/behaviors/forms/interaction
Request Body:
{
  "sessionId": "sess_123456789",
  "formId": "approval-form-001",
  "interactions": [
    {
      "fieldName": "approver_comments",
      "actionType": "FOCUS",
      "timestamp": "2025-07-23T10:30:00Z",
      "fieldValue": "[HASHED]",
      "validationErrors": []
    },
    {
      "fieldName": "approval_decision",
      "actionType": "CHANGE",
      "timestamp": "2025-07-23T10:30:15Z",
      "previousValue": "[HASHED]",
      "newValue": "[HASHED]"
    }
  ]
}
```

#### 4. Session Flow Analytics
```json
GET /api/v1/analytics/session-flows/{sessionId}
Headers: {
  "Authorization": "Bearer {jwt_token}"
}

Response:
{
  "sessionId": "sess_123456789",
  "totalDuration": 1847000,
  "pageFlow": [
    {
      "pageUrl": "/dashboard",
      "entryTime": "2025-07-23T10:00:00Z",
      "exitTime": "2025-07-23T10:05:30Z",
      "timeOnPage": 330000,
      "interactions": 15,
      "scrollDepth": 85
    },
    {
      "pageUrl": "/documents/pending",
      "entryTime": "2025-07-23T10:05:30Z",
      "exitTime": "2025-07-23T10:25:15Z",
      "timeOnPage": 1185000,
      "interactions": 47,
      "scrollDepth": 100
    }
  ],
  "behaviorSummary": {
    "totalClicks": 62,
    "totalScrolls": 23,
    "totalFormInteractions": 8,
    "averageTimePerPage": 757500,
    "bounceRate": 0.15
  }
}
```

#### 5. Real-time Dashboard Data
```json
GET /api/v1/dashboard/realtime/behavior-analytics
Headers: {
  "Authorization": "Bearer {jwt_token}"
}

Response:
{
  "currentActiveUsers": 1247,
  "peakToday": 2105,
  "liveInteractions": {
    "clicksPerMinute": 1850,
    "activePages": 45,
    "formSubmissions": 23,
    "documentAccesses": 156
  },
  "behaviorMetrics": {
    "averageSessionDuration": 1680000,
    "averageClicksPerSession": 47,
    "averagePageViews": 8.3,
    "topInteractionTypes": [
      {"type": "CLICK", "count": 15670},
      {"type": "SCROLL", "count": 8920},
      {"type": "FORM_INTERACTION", "count": 3450}
    ]
  },
  "systemHealth": {
    "behaviorProcessingLag": 85,
    "dataQualityScore": 98.7,
    "errorRate": 0.03
  },
  "alerts": [
    {
      "type": "behavior_anomaly",
      "message": "Unusual click pattern detected on approval page",
      "severity": "medium",
      "timestamp": "2025-07-23T10:25:00Z",
      "affectedUsers": 12
    }
  ]
}
```

#### 6. User Journey Analysis
```json
GET /api/v1/analytics/user-journey
Query Params: ?userId=user_hash_123&dateRange=7d&includeDetails=true

Response:
{
  "userHash": "user_hash_123",
  "analysisperiod": "2025-07-16T00:00:00Z to 2025-07-23T23:59:59Z",
  "journeyMetrics": {
    "totalSessions": 18,
    "averageSessionDuration": 1456000,
    "totalInteractions": 847,
    "uniquePagesVisited": 23,
    "conversionEvents": 5
  },
  "commonPaths": [
    {
      "path": "/login → /dashboard → /documents/pending → /documents/review",
      "frequency": 12,
      "averageDuration": 890000,
      "conversionRate": 0.83
    }
  ],
  "behaviorPatterns": {
    "peakActivityHours": ["09:00-11:00", "14:00-16:00"],
    "preferredInteractionStyle": "methodical_reviewer",
    "documentAccessPatterns": "sequential_reader",
    "formCompletionRate": 0.94
  }
}
```

---

## Security Architecture

### Authentication & Authorization
```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   E-Office      │    │  ACTIVITY-      │    │    Oracle       │
│   SSO/LDAP      │───▶│  TRACKING       │───▶│   Database      │
│                 │    │   Gateway       │    │                 │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│ JWT Token       │    │  RBAC Engine    │    │ Encrypted       │
│ (15 min TTL)    │    │  Role Mapping   │    │ Data at Rest    │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Data Privacy Implementation
**Data Anonymization Strategy:**
```java
// User ID Hashing
String userHash = SHA256.hash(userId + salt + date);

// IP Address Masking
String ipMask = ipAddress.substring(0, ipAddress.lastIndexOf('.')) + ".0";
String ipHash = SHA256.hash(ipMask + salt);

// Document Reference Hashing
String docHash = SHA256.hash(documentId + salt);
```

**Encryption at Rest:**
- Oracle TDE (Transparent Data Encryption)
- Application-level encryption for sensitive metadata
- Key rotation every 90 days

**Encryption in Transit:**
- TLS 1.3 for all API communications
- Certificate pinning for mobile clients
- HSTS headers for web security

### Compliance Measures
**PDPA Compliance:**
- No PII collection in activity logs
- Data retention automated cleanup
- User consent tracking (if required)
- Data export/deletion capabilities

**Audit Trail:**
- All admin actions logged
- Immutable audit logs
- Digital signatures for critical events
- Compliance reporting automation

---

## Implementation Strategy

### Development Approach
**Phase 1: Core Infrastructure (4 weeks)**
- Database setup and schema creation
- API Gateway implementation
- Basic authentication integration
- Docker containerization

**Phase 2: Core Tracking Services (6 weeks)**
- User authentication tracking
- Document access tracking
- Basic activity logging
- Real-time data pipeline

**Phase 3: Advanced Features (4 weeks)**
- Approval process tracking
- Module usage analytics
- Real-time dashboard
- Alert system

**Phase 4: Reporting & Analytics (4 weeks)**
- Report generation engine
- Dashboard enhancements
- Data export capabilities
- Performance optimization

**Phase 5: Testing & Deployment (3 weeks)**
- Integration testing
- Performance testing
- Security testing
- Production deployment

### Deployment Strategy
**Environment Setup:**
```
Development → Staging → Production
    ↓           ↓          ↓
  Docker     Docker    Docker
   Local     VM/Cloud   Dedicated
                        Servers
```

**CI/CD Pipeline:**
1. **Code Commit** → Git Repository
2. **Build Stage** → Maven/Gradle build + Docker image
3. **Test Stage** → Unit tests + Integration tests
4. **Deploy to Staging** → Automated deployment
5. **Smoke Tests** → Basic functionality verification
6. **Deploy to Production** → Blue-green deployment

### Testing Strategy
**Unit Testing:** 80%+ code coverage
- JUnit 5 for Java services
- Mockito for dependency mocking
- TestContainers for database testing

**Integration Testing:**
- API endpoint testing with REST Assured
- Database integration testing
- Message queue integration testing

**Performance Testing:**
- JMeter for load testing
- 4,000 concurrent users simulation
- Database performance benchmarking

**Security Testing:**
- OWASP ZAP for vulnerability scanning
- Penetration testing for critical paths
- Compliance audit preparation

---

## Risk Assessment & Mitigation

### Technical Risks

#### R1: Database Performance Under Load
**Risk Level:** High  
**Mitigation:**
- Implement database connection pooling
- Use Oracle partitioning for large tables
- Regular performance monitoring and tuning
- Read replicas for reporting queries

#### R2: Real-time Processing Bottlenecks
**Risk Level:** Medium  
**Mitigation:**
- Kafka partitioning strategy
- Horizontal scaling of processing services
- Circuit breaker patterns
- Graceful degradation mechanisms

#### R3: Oracle Integration Complexity
**Risk Level:** Medium  
**Mitigation:**
- Early prototype development
- Oracle DBA consultation
- Comprehensive testing environment
- Fallback to PostgreSQL if needed

### Business Risks

#### R4: PDPA Compliance Gaps
**Risk Level:** Critical  
**Mitigation:**
- Legal review of data handling
- Privacy impact assessment
- Regular compliance audits
- Staff training on data protection

#### R5: User Adoption Resistance
**Risk Level:** Low  
**Mitigation:**
- Transparent communication about benefits
- Minimal performance impact design
- Role-based access to reduce concerns
- Gradual rollout with feedback incorporation

---

## Monitoring & Operations

### System Monitoring
**Application Metrics:**
- Response time percentiles (P50, P95, P99)
- Error rates by endpoint
- Active user counts
- Database query performance

**Infrastructure Metrics:**
- CPU and memory utilization
- Database connection pool stats
- Kafka lag monitoring
- Cache hit ratios

**Business Metrics:**
- Activity tracking accuracy
- Report generation times
- Dashboard load performance
- User engagement patterns

### Alerting Strategy
**Critical Alerts:**
- System downtime (> 30 seconds)
- Database connection failures
- High error rates (> 5%)
- Security incidents

**Warning Alerts:**
- High response times (> 2 seconds)
- Low cache hit ratios (< 80%)
- Unusual user patterns
- Resource utilization (> 80%)

### Operational Procedures
**Daily Operations:**
- Health check monitoring
- Performance metrics review
- Error log analysis
- Backup verification

**Weekly Operations:**
- Capacity planning review
- Security log analysis
- Performance optimization
- Documentation updates

**Monthly Operations:**
- Compliance reporting
- Capacity forecasting
- Security assessments
- Disaster recovery testing

---

## Performance Optimization

### High-Volume Behavior Data Processing
**Streaming Data Architecture:**
```yaml
# Kafka Configuration for Behavior Events
kafka:
  behavior-events:
    topic: user-behaviors
    partitions: 24
    replication-factor: 3
    batch-size: 16384
    linger-ms: 10
    compression-type: lz4
    max-request-size: 1048576
```

**Batch Processing Configuration:**
```java
// Behavior event batch processing
@KafkaListener(topics = "user-behaviors", 
               containerFactory = "behaviorEventBatchListenerFactory")
public void processBehaviorEventsBatch(List<BehaviorEvent> events) {
    // Process 1000 events per batch for optimal performance
    behaviorService.processBatchEvents(events);
}

// Async bulk database inserts
@Async("behaviorTaskExecutor")
@Transactional
public CompletableFuture<Void> bulkInsertBehaviors(List<BehaviorEvent> events) {
    jdbcTemplate.batchUpdate(INSERT_BEHAVIOR_SQL, events);
    return CompletableFuture.completedFuture(null);
}
```

### Database Optimization for Behavior Tracking
**Partitioning Strategy:**
```sql
-- Monthly partitioning for behavior data
CREATE TABLE user_behaviors (
    behavior_id NUMBER GENERATED BY DEFAULT AS IDENTITY,
    session_id VARCHAR2(64) NOT NULL,
    event_type VARCHAR2(50) NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    element_id VARCHAR2(255),
    coordinates_x NUMBER,
    coordinates_y NUMBER,
    metadata_json CLOB
) 
PARTITION BY RANGE (timestamp) (
    PARTITION behaviors_202501 VALUES LESS THAN (DATE '2025-02-01'),
    PARTITION behaviors_202502 VALUES LESS THAN (DATE '2025-03-01'),
    PARTITION behaviors_202503 VALUES LESS THAN (DATE '2025-04-01')
);

-- Optimized indexes for behavior queries
CREATE INDEX idx_behaviors_session_time 
ON user_behaviors (session_id, timestamp) LOCAL;

CREATE INDEX idx_behaviors_type_time 
ON user_behaviors (event_type, timestamp) LOCAL;

CREATE BITMAP INDEX idx_behaviors_element_type 
ON user_behaviors (element_type) LOCAL;
```

**Query Optimization for Real-time Analytics:**
```sql
-- Materialized view for real-time behavior metrics
CREATE MATERIALIZED VIEW behavior_realtime_summary
BUILD IMMEDIATE
REFRESH FAST ON COMMIT
AS SELECT 
    TRUNC(timestamp, 'MI') as minute_bucket,
    event_type,
    COUNT(*) as event_count,
    COUNT(DISTINCT session_id) as unique_sessions
FROM user_behaviors 
WHERE timestamp >= SYSDATE - 1/24  -- Last hour
GROUP BY TRUNC(timestamp, 'MI'), event_type;

-- Fast session flow queries
CREATE MATERIALIZED VIEW session_flow_cache
REFRESH FAST ON DEMAND
AS SELECT 
    session_id,
    LISTAGG(page_url, ' → ') WITHIN GROUP (ORDER BY timestamp) as user_flow,
    COUNT(*) as page_views,
    MAX(timestamp) - MIN(timestamp) as session_duration
FROM page_views 
WHERE timestamp >= SYSDATE - 7  -- Last 7 days
GROUP BY session_id;
```

### Caching Strategy for Behavior Analytics
**Multi-Layer Caching:**
```java
// L1: Application Cache (Caffeine) - Ultra fast access
@Cacheable(value = "behavior-summary", key = "#sessionId")
public BehaviorSummary getSessionBehaviorSummary(String sessionId) {
    return behaviorRepository.calculateSessionSummary(sessionId);
}

// L2: Redis Cache - Shared across instances
@Cacheable(value = "realtime-metrics", key = "#timeWindow")
public RealtimeMetrics getRealtimeBehaviorMetrics(String timeWindow) {
    return metricsService.calculateRealtimeMetrics(timeWindow);
}

// L3: Database Materialized Views - Pre-aggregated data
public DashboardData getDashboardData(String department) {
    return jdbcTemplate.queryForObject(DASHBOARD_MV_QUERY, 
                                      DashboardData.class, department);
}
```

**Cache Configuration:**
```yaml
# Redis Configuration for Behavior Data
redis:
  behavior-cache:
    cluster-nodes: redis-cluster-01:6379,redis-cluster-02:6379,redis-cluster-03:6379
    max-redirects: 3
    timeout: 2000ms
    lettuce:
      pool:
        max-active: 20
        max-idle: 10
        min-idle: 5
      cluster:
        refresh:
          adaptive: true
          period: 30s

# Caffeine L1 Cache
caffeine:
  behavior-summary:
    maximum-size: 10000
    expire-after-write: 5m
  user-flows:
    maximum-size: 5000
    expire-after-write: 10m
```

### API Performance for High-Frequency Events
**Event Buffering and Batching:**
```javascript
// Client-side event buffering
class BehaviorTracker {
    constructor() {
        this.eventBuffer = [];
        this.bufferSize = 50;
        this.flushInterval = 5000; // 5 seconds
        this.setupAutoFlush();
    }
    
    trackEvent(event) {
        this.eventBuffer.push({
            ...event,
            timestamp: new Date().toISOString(),
            sessionId: this.sessionId
        });
        
        if (this.eventBuffer.length >= this.bufferSize) {
            this.flushEvents();
        }
    }
    
    async flushEvents() {
        if (this.eventBuffer.length === 0) return;
        
        const events = [...this.eventBuffer];
        this.eventBuffer = [];
        
        try {
            await fetch('/api/v1/behaviors/events/bulk', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({ events })
            });
        } catch (error) {
            console.error('Failed to send behavior events:', error);
            // Retry logic here
        }
    }
}
```

**Server-side Rate Limiting:**
```java
@RestController
@RateLimiter(name = "behavior-tracking", fallbackMethod = "rateLimitFallback")
public class BehaviorTrackingController {
    
    @PostMapping("/api/v1/behaviors/events/bulk")
    @RateLimiter(name = "behavior-bulk", 
                 fallbackMethod = "bulkEventsFallback")
    public ResponseEntity<EventResponse> trackBehaviorEvents(
            @RequestBody @Valid BulkEventRequest request) {
        
        // Async processing to avoid blocking
        CompletableFuture<Void> processing = 
            behaviorService.processEventsAsync(request.getEvents());
            
        return ResponseEntity.accepted()
                .body(new EventResponse("ACCEPTED", request.getEvents().size()));
    }
    
    public ResponseEntity<EventResponse> bulkEventsFallback(
            BulkEventRequest request, Exception ex) {
        // Fallback: Queue events for later processing
        eventQueueService.queueEvents(request.getEvents());
        return ResponseEntity.status(429)
                .body(new EventResponse("RATE_LIMITED", 0));
    }
}
```

### Connection Pooling for High Concurrency
**Database Connection Configuration:**
```yaml
# HikariCP Configuration for High-Volume Behavior Data
hikari:
  behavior-db:
    maximum-pool-size: 50
    minimum-idle: 20
    connection-timeout: 10000
    idle-timeout: 300000
    max-lifetime: 900000
    leak-detection-threshold: 30000
    auto-commit: false
    
# Separate pool for analytics queries
analytics-db:
  maximum-pool-size: 20
  minimum-idle: 5
  connection-timeout: 30000
  validation-timeout: 5000
  read-only: true
```

### Performance Monitoring
**Key Performance Indicators:**
```java
@Component
public class BehaviorTrackingMetrics {
    
    @EventListener
    public void onBehaviorEventProcessed(BehaviorEventProcessedEvent event) {
        // Track processing latency
        Timer.Sample sample = Timer.start(meterRegistry);
        sample.stop(Timer.builder("behavior.processing.time")
                   .tag("event.type", event.getEventType())
                   .register(meterRegistry));
                   
        // Track throughput
        Counter.builder("behavior.events.processed")
               .tag("event.type", event.getEventType())
               .register(meterRegistry)
               .increment();
    }
    
    @Scheduled(fixedRate = 60000) // Every minute
    public void recordSystemMetrics() {
        // Buffer size monitoring
        Gauge.builder("behavior.buffer.size")
             .register(meterRegistry, eventBuffer, Collection::size);
             
        // Database connection pool usage
        Gauge.builder("behavior.db.connections.active")
             .register(meterRegistry, hikariDataSource, 
                      HikariDataSource::getHikariPoolMXBean)
             .tag("pool", "behavior-db");
    }
}
```

---

## Future Enhancements

### Short-term Enhancements (6 months)
- Machine learning for anomaly detection
- Advanced analytics and predictions
- Mobile application support
- API versioning and backward compatibility

### Long-term Enhancements (12+ months)
- Multi-tenant architecture support
- Advanced compliance reporting
- Integration with external SIEM systems
- Blockchain-based audit trail

---

## Conclusion

This comprehensive design document outlines the technical architecture for the **ACTIVITY-TRACKING system**, specifically engineered to capture and analyze detailed user behavior patterns in the E-Office environment. The system is designed to track **ทุกการกดปุ่ม, flow การใช้งานระบบในแต่ละ session รวมถึงทุกอย่างที่สามารถเก็บได้จากพฤติกรรมของ user** (every button click, system usage flows in each session, including everything that can be captured from user behavior).

### Key Design Principles Implemented:

**1. Comprehensive Behavior Capture:**
- Click-level tracking with precise coordinate and timing data
- Mouse movement patterns and hover interactions
- Form field interactions including validation errors
- Scroll behaviors with depth and speed metrics
- Page navigation flows and session journey mapping
- Real-time error and performance tracking

**2. High-Performance Architecture:**
- Event-driven architecture with Kafka streaming for 100,000+ events/hour
- Multi-layer caching strategy (Caffeine L1 + Redis L2 + DB materialized views)
- Database partitioning strategy supporting 800GB annual growth
- Async processing pipeline minimizing user experience impact
- Optimized for 2,000-8,000 concurrent users with sub-100ms response times

**3. Advanced Analytics Capabilities:**
- Real-time behavior pattern analysis and anomaly detection
- User journey visualization with conversion funnel tracking
- Heat map generation for page interaction analysis
- Session replay capabilities for detailed behavior reconstruction
- Predictive analytics for user experience optimization

**4. Enterprise Security & Compliance:**
- PDPA-compliant data anonymization with SHA-256 hashing
- Privacy-by-design architecture with configurable data retention
- Oracle TDE encryption at rest with role-based access controls
- Audit trail for all behavior data access and modifications

**5. Scalable Technology Stack:**
- **Backend:** Java Spring Boot with Java 1.8→18 migration path
- **Database:** Oracle 19c+ with advanced partitioning and indexing
- **Frontend:** React 18+ with TypeScript and comprehensive tracking SDK
- **Caching:** Redis Cluster for distributed session management
- **Messaging:** Apache Kafka for real-time event streaming
- **Containerization:** Docker with Kubernetes orchestration

### Business Value Delivery:

The ACTIVITY-TRACKING system delivers unprecedented insights into user behavior patterns, enabling data-driven decisions for:
- **User Experience Optimization:** Identify friction points and optimize workflows
- **Security Enhancement:** Detect unusual behavior patterns and potential threats  
- **Compliance Reporting:** Generate detailed audit trails for regulatory requirements
- **Performance Monitoring:** Track system usage patterns and capacity planning
- **Training Insights:** Understand user learning patterns and knowledge gaps

### Implementation Readiness:

The design supports incremental 21-week development across 5 major epics, with clear dependencies and risk mitigation strategies. The modular architecture enables parallel development teams while maintaining integration consistency.

This technical blueprint provides the foundation for building a world-class user behavior analytics platform that transforms how the E-Office understands and optimizes user interactions with their digital systems.

---

**Document Status:** Technical Design Complete  
**Last Updated:** January 23, 2025  
**Implementation Start:** February 2025  
**Target Completion:** July 2025  
**Approved By:** [Pending Architecture Review Board]
