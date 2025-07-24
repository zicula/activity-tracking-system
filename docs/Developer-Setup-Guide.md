# Developer Setup Guide - ACTIVITY-TRACKING System

## Document Information
**Project:** ACTIVITY-TRACKING System  
**Version:** 1.0  
**Date:** December 2024  
**Technology Stack:** Java JDK 1.8, JSF 2.x, PrimeFaces 8.x+, ELK Stack  
**Target Audience:** Development Team

---

## 🚀 Quick Start Guide

### Prerequisites Checklist
- [ ] Java JDK 1.8 installed
- [ ] Maven 3.6+ installed
- [ ] Docker Desktop installed and running
- [ ] Git configured with SSH keys
- [ ] IDE configured (IntelliJ IDEA recommended)
- [ ] Access to company VPN (for production log access)

### 5-Minute Setup
```bash
# 1. Clone the repository
git clone git@github.com:zicula/activity-tracking-system.git
cd activity-tracking-system

# 2. Start development environment
docker-compose up -d

# 3. Build the application
mvn clean install

# 4. Run the application
mvn spring-boot:run

# 5. Open browser
open http://localhost:8080
```

---

## 📋 Detailed Development Environment Setup

### 1. Java Development Kit 1.8

#### Installation (macOS)
```bash
# Using Homebrew
brew install openjdk@8

# Add to PATH
echo 'export PATH="/opt/homebrew/opt/openjdk@8/bin:$PATH"' >> ~/.zshrc
echo 'export JAVA_HOME="/opt/homebrew/opt/openjdk@8"' >> ~/.zshrc
source ~/.zshrc

# Verify installation
java -version
javac -version
```

#### Installation (Ubuntu/Linux)
```bash
# Install OpenJDK 8
sudo apt update
sudo apt install openjdk-8-jdk

# Set JAVA_HOME
echo 'export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64' >> ~/.bashrc
echo 'export PATH=$JAVA_HOME/bin:$PATH' >> ~/.bashrc
source ~/.bashrc

# Verify installation
java -version
```

#### Installation (Windows)
```powershell
# Using Chocolatey
choco install openjdk8

# Or download from Oracle/OpenJDK
# Set JAVA_HOME environment variable
# Add %JAVA_HOME%\bin to PATH
```

### 2. Apache Maven 3.6+

#### Installation (macOS)
```bash
# Using Homebrew
brew install maven

# Verify installation
mvn -version
```

#### Installation (Ubuntu/Linux)
```bash
# Install Maven
sudo apt install maven

# Verify installation
mvn -version
```

#### Maven Settings Configuration
```xml
<!-- ~/.m2/settings.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 
                              http://maven.apache.org/xsd/settings-1.0.0.xsd">
  
  <localRepository>${user.home}/.m2/repository</localRepository>
  
  <profiles>
    <profile>
      <id>development</id>
      <properties>
        <elasticsearch.host>localhost</elasticsearch.host>
        <elasticsearch.port>9200</elasticsearch.port>
        <logstash.host>localhost</logstash.host>
        <logstash.port>5000</logstash.port>
      </properties>
    </profile>
  </profiles>
  
  <activeProfiles>
    <activeProfile>development</activeProfile>
  </activeProfiles>
  
</settings>
```

### 3. Docker Development Environment

#### Docker Compose Configuration
```yaml
# docker-compose.yml
version: '3.8'

services:
  # Elasticsearch for log storage and search
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    container_name: activity-tracking-elasticsearch
    environment:
      - node.name=elasticsearch
      - cluster.name=activity-tracking-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - elasticsearch_data:/usr/share/elasticsearch/data
    ports:
      - "9200:9200"
      - "9300:9300"
    networks:
      - activity-tracking-network

  # Logstash for log processing
  logstash:
    image: docker.elastic.co/logstash/logstash:7.15.0
    container_name: activity-tracking-logstash
    volumes:
      - ./logstash/config/logstash.yml:/usr/share/logstash/config/logstash.yml:ro
      - ./logstash/pipeline:/usr/share/logstash/pipeline:ro
      - ./logs:/var/log/applications:ro
    ports:
      - "5044:5044"
      - "5000:5000/tcp"
      - "5000:5000/udp"
      - "9600:9600"
    environment:
      LS_JAVA_OPTS: "-Xmx512m -Xms512m"
    networks:
      - activity-tracking-network
    depends_on:
      - elasticsearch

  # Kibana for data visualization (development only)
  kibana:
    image: docker.elastic.co/kibana/kibana:7.15.0
    container_name: activity-tracking-kibana
    ports:
      - "5601:5601"
    environment:
      ELASTICSEARCH_URL: http://elasticsearch:9200
      ELASTICSEARCH_HOSTS: '["http://elasticsearch:9200"]'
    networks:
      - activity-tracking-network
    depends_on:
      - elasticsearch

  # PostgreSQL for application metadata (if needed)
  postgres:
    image: postgres:13-alpine
    container_name: activity-tracking-postgres
    environment:
      POSTGRES_DB: activity_tracking_dev
      POSTGRES_USER: dev_user
      POSTGRES_PASSWORD: dev_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init:/docker-entrypoint-initdb.d:ro
    ports:
      - "5432:5432"
    networks:
      - activity-tracking-network

  # Redis for caching and session storage
  redis:
    image: redis:7-alpine
    container_name: activity-tracking-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - activity-tracking-network

volumes:
  elasticsearch_data:
    driver: local
  postgres_data:
    driver: local
  redis_data:
    driver: local

networks:
  activity-tracking-network:
    driver: bridge
```

#### Starting Development Environment
```bash
# Start all services
docker-compose up -d

# Check service status
docker-compose ps

# View logs
docker-compose logs -f elasticsearch
docker-compose logs -f logstash

# Stop services
docker-compose down

# Reset data (WARNING: deletes all data)
docker-compose down -v
```

### 4. IDE Configuration (IntelliJ IDEA)

#### Project Import
```bash
# Open IntelliJ IDEA
# File → Open → Select project root directory
# Choose "Import project from external model" → Maven
# Use default settings and finish import
```

#### Code Style Configuration
```xml
<!-- IntelliJ Code Style Settings -->
<!-- File → Settings → Editor → Code Style → Java -->
<code_scheme name="ActivityTrackingStyle" version="173">
  <JavaCodeStyleSettings>
    <option name="LAYOUT_STATIC_IMPORTS_SEPARATELY" value="true" />
    <option name="USE_FQ_CLASS_NAMES" value="false" />
    <option name="INSERT_INNER_CLASS_IMPORTS" value="true" />
    <option name="CLASS_COUNT_TO_USE_IMPORT_ON_DEMAND" value="999" />
    <option name="NAMES_COUNT_TO_USE_IMPORT_ON_DEMAND" value="999" />
  </JavaCodeStyleSettings>
  <codeStyleSettings language="JAVA">
    <option name="KEEP_LINE_BREAKS" value="false" />
    <option name="KEEP_FIRST_COLUMN_COMMENT" value="false" />
    <option name="BRACE_STYLE" value="END_OF_LINE" />
    <option name="CLASS_BRACE_STYLE" value="END_OF_LINE" />
    <option name="METHOD_BRACE_STYLE" value="END_OF_LINE" />
    <option name="ELSE_ON_NEW_LINE" value="false" />
    <option name="WHILE_ON_NEW_LINE" value="false" />
    <option name="CATCH_ON_NEW_LINE" value="false" />
    <option name="FINALLY_ON_NEW_LINE" value="false" />
    <option name="ALIGN_MULTILINE_PARAMETERS" value="false" />
    <option name="SPACE_WITHIN_METHOD_CALL_PARENTHESES" value="false" />
    <option name="SPACE_WITHIN_METHOD_PARENTHESES" value="false" />
    <option name="SPACE_WITHIN_IF_PARENTHESES" value="false" />
    <option name="SPACE_WITHIN_WHILE_PARENTHESES" value="false" />
    <option name="SPACE_WITHIN_FOR_PARENTHESES" value="false" />
    <option name="SPACE_WITHIN_TRY_PARENTHESES" value="false" />
    <option name="SPACE_WITHIN_CATCH_PARENTHESES" value="false" />
    <option name="SPACE_WITHIN_SWITCH_PARENTHESES" value="false" />
    <option name="SPACE_WITHIN_SYNCHRONIZED_PARENTHESES" value="false" />
    <option name="SPACE_WITHIN_CAST_PARENTHESES" value="false" />
    <option name="SPACE_WITHIN_BRACKETS" value="false" />
    <option name="SPACE_WITHIN_BRACES" value="true" />
    <option name="SPACE_WITHIN_ARRAY_INITIALIZER_BRACES" value="true" />
  </codeStyleSettings>
</code_scheme>
```

#### Useful Plugins
```
Required Plugins:
- Lombok Plugin (for @Data, @Slf4j annotations)
- Maven Helper (for dependency management)
- SonarLint (for code quality)
- Docker Integration (for container management)

Optional but Recommended:
- Git Integration (usually pre-installed)
- Database Navigator (for database management)
- Rainbow Brackets (for code readability)
- Key Promoter X (for learning shortcuts)
```

#### Run Configurations
```xml
<!-- Application Run Configuration -->
<component name="ProjectRunConfigurationManager">
  <configuration default="false" name="ActivityTrackingApplication" type="SpringBootApplicationConfigurationType">
    <option name="ACTIVE_PROFILES" value="development" />
    <option name="MAIN_CLASS_NAME" value="com.eoffice.activitytracking.ActivityTrackingApplication" />
    <option name="MODULE_NAME" value="activity-tracking-main" />
    <option name="VM_PARAMETERS" value="-Xmx2g -Xms1g -Dspring.profiles.active=development" />
    <option name="WORKING_DIRECTORY" value="$MODULE_WORKING_DIR$" />
    <envs>
      <env name="ELASTICSEARCH_HOST" value="localhost" />
      <env name="ELASTICSEARCH_PORT" value="9200" />
      <env name="LOG_LEVEL" value="DEBUG" />
    </envs>
    <method v="2">
      <option name="Make" enabled="true" />
    </method>
  </configuration>
</component>
```

---

## 🏗️ Project Structure and Architecture

### Directory Structure
```
activity-tracking-system/
├── README.md
├── docker-compose.yml
├── pom.xml
├── .gitignore
├── .editorconfig
│
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/eoffice/activitytracking/
│   │   │       ├── ActivityTrackingApplication.java
│   │   │       ├── config/                    # Configuration classes
│   │   │       │   ├── ElasticsearchConfig.java
│   │   │       │   ├── SecurityConfig.java
│   │   │       │   └── WebConfig.java
│   │   │       ├── controller/                # JSF Managed Beans
│   │   │       │   ├── DashboardController.java
│   │   │       │   ├── AuthenticationController.java
│   │   │       │   └── ReportController.java
│   │   │       ├── service/                   # Business Logic Layer
│   │   │       │   ├── LogAnalysisService.java
│   │   │       │   ├── ElasticsearchService.java
│   │   │       │   ├── SecurityService.java
│   │   │       │   └── ReportingService.java
│   │   │       ├── repository/                # Data Access Layer
│   │   │       │   ├── ElasticsearchRepository.java
│   │   │       │   └── LogEntryRepository.java
│   │   │       ├── model/                     # Data Models
│   │   │       │   ├── LogEntry.java
│   │   │       │   ├── ActivitySummary.java
│   │   │       │   ├── UserSession.java
│   │   │       │   └── SystemMetrics.java
│   │   │       ├── dto/                       # Data Transfer Objects
│   │   │       │   ├── DashboardStats.java
│   │   │       │   ├── AuthenticationStats.java
│   │   │       │   └── PerformanceMetrics.java
│   │   │       ├── util/                      # Utility Classes
│   │   │       │   ├── LogPatternMatcher.java
│   │   │       │   ├── DateUtils.java
│   │   │       │   └── SecurityUtils.java
│   │   │       └── exception/                 # Custom Exceptions
│   │   │           ├── LogProcessingException.java
│   │   │           ├── ElasticsearchException.java
│   │   │           └── SecurityException.java
│   │   │
│   │   ├── resources/
│   │   │   ├── application.yml                # Main configuration
│   │   │   ├── application-development.yml    # Development config
│   │   │   ├── application-production.yml     # Production config
│   │   │   ├── logback-spring.xml            # Logging configuration
│   │   │   ├── static/                       # Static web resources
│   │   │   │   ├── css/
│   │   │   │   ├── js/
│   │   │   │   └── images/
│   │   │   └── templates/                    # Thymeleaf templates (if used)
│   │   │
│   │   └── webapp/                           # JSF Web Application
│   │       ├── WEB-INF/
│   │       │   ├── web.xml                   # Web application descriptor
│   │       │   ├── faces-config.xml          # JSF configuration
│   │       │   └── templates/                # JSF templates
│   │       │       ├── main-template.xhtml
│   │       │       └── dashboard-template.xhtml
│   │       ├── pages/                        # JSF pages
│   │       │   ├── dashboard/
│   │       │   │   ├── overview.xhtml
│   │       │   │   ├── authentication.xhtml
│   │       │   │   ├── documents.xhtml
│   │       │   │   └── performance.xhtml
│   │       │   ├── reports/
│   │       │   │   ├── daily-report.xhtml
│   │       │   │   └── user-report.xhtml
│   │       │   └── admin/
│   │       │       ├── log-monitoring.xhtml
│   │       │       └── system-config.xhtml
│   │       └── resources/                    # JSF resources
│   │           ├── css/
│   │           ├── js/
│   │           └── images/
│   │
│   └── test/
│       ├── java/
│       │   └── com/eoffice/activitytracking/
│       │       ├── service/                  # Service tests
│       │       ├── controller/               # Controller tests
│       │       ├── integration/              # Integration tests
│       │       └── util/                     # Utility tests
│       └── resources/
│           ├── application-test.yml          # Test configuration
│           ├── test-data/                    # Test data files
│           └── sample-logs/                  # Sample log files
│
├── docs/                                     # Documentation
│   ├── api/                                  # API documentation
│   ├── deployment/                           # Deployment guides
│   ├── architecture/                         # Architecture diagrams
│   └── user-guides/                          # User manuals
│
├── scripts/                                  # Utility scripts
│   ├── setup-dev-environment.sh             # Development setup
│   ├── deploy-to-staging.sh                 # Staging deployment
│   └── backup-elasticsearch.sh              # Backup scripts
│
├── logstash/                                 # Logstash configuration
│   ├── config/
│   │   └── logstash.yml
│   └── pipeline/
│       ├── production-logs.conf
│       └── development-logs.conf
│
├── database/                                 # Database scripts
│   ├── init/                                 # Initialization scripts
│   └── migrations/                           # Migration scripts
│
└── docker/                                   # Docker configurations
    ├── Dockerfile                            # Application container
    ├── Dockerfile.dev                        # Development container
    └── nginx/                                # Nginx configuration
        └── nginx.conf
```

### Maven Project Configuration (pom.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
                             http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.eoffice</groupId>
    <artifactId>activity-tracking-system</artifactId>
    <version>1.0.0-SNAPSHOT</version>
    <packaging>war</packaging>
    <name>Activity Tracking System</name>
    <description>Activity Tracking System for E-Office Applications</description>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        
        <!-- Dependency Versions -->
        <spring-boot.version>2.7.18</spring-boot.version>
        <jsf.version>2.3.17</jsf.version>
        <primefaces.version>12.0.0</primefaces.version>
        <elasticsearch.version>7.15.2</elasticsearch.version>
        <lombok.version>1.18.30</lombok.version>
        <junit.version>5.9.3</junit.version>
        <mockito.version>4.11.0</mockito.version>
        <testcontainers.version>1.19.3</testcontainers.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-dependencies</artifactId>
                <version>${spring-boot.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- Spring Boot Starters -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-security</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-validation</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>

        <!-- JSF Dependencies -->
        <dependency>
            <groupId>org.apache.myfaces.core</groupId>
            <artifactId>myfaces-impl</artifactId>
            <version>${jsf.version}</version>
        </dependency>
        
        <dependency>
            <groupId>org.apache.myfaces.core</groupId>
            <artifactId>myfaces-api</artifactId>
            <version>${jsf.version}</version>
        </dependency>

        <!-- PrimeFaces -->
        <dependency>
            <groupId>org.primefaces</groupId>
            <artifactId>primefaces</artifactId>
            <version>${primefaces.version}</version>
        </dependency>

        <!-- Elasticsearch -->
        <dependency>
            <groupId>org.elasticsearch.client</groupId>
            <artifactId>elasticsearch-rest-high-level-client</artifactId>
            <version>${elasticsearch.version}</version>
        </dependency>

        <!-- Utility Libraries -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>${lombok.version}</version>
            <scope>provided</scope>
        </dependency>
        
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
        </dependency>
        
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>

        <!-- Testing Dependencies -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.junit.jupiter</groupId>
            <artifactId>junit-jupiter</artifactId>
            <version>${junit.version}</version>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.mockito</groupId>
            <artifactId>mockito-core</artifactId>
            <version>${mockito.version}</version>
            <scope>test</scope>
        </dependency>
        
        <dependency>
            <groupId>org.testcontainers</groupId>
            <artifactId>elasticsearch</artifactId>
            <version>${testcontainers.version}</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <finalName>activity-tracking-system</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>${spring-boot.version}</version>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
            
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.11.0</version>
                <configuration>
                    <source>8</source>
                    <target>8</target>
                    <encoding>UTF-8</encoding>
                </configuration>
            </plugin>
            
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-war-plugin</artifactId>
                <version>3.3.2</version>
                <configuration>
                    <failOnMissingWebXml>false</failOnMissingWebXml>
                </configuration>
            </plugin>
            
            <!-- Code Quality Plugins -->
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
                </executions>
            </plugin>
            
            <plugin>
                <groupId>org.sonarsource.scanner.maven</groupId>
                <artifactId>sonar-maven-plugin</artifactId>
                <version>3.9.1.2184</version>
            </plugin>
        </plugins>
    </build>

    <profiles>
        <profile>
            <id>development</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <spring.profiles.active>development</spring.profiles.active>
            </properties>
        </profile>
        
        <profile>
            <id>production</id>
            <properties>
                <spring.profiles.active>production</spring.profiles.active>
            </properties>
        </profile>
    </profiles>
</project>
```

---

## ⚙️ Configuration Files

### Application Configuration (application.yml)
```yaml
# Main application configuration
spring:
  application:
    name: activity-tracking-system
  
  profiles:
    active: ${SPRING_PROFILES_ACTIVE:development}
  
  web:
    resources:
      static-locations: classpath:/static/,classpath:/META-INF/resources/
  
  servlet:
    multipart:
      max-file-size: 10MB
      max-request-size: 10MB

# Server configuration
server:
  port: ${SERVER_PORT:8080}
  servlet:
    context-path: /activity-tracking
    session:
      timeout: 30m
  
  compression:
    enabled: true
    mime-types: text/html,text/xml,text/plain,text/css,text/javascript,application/javascript,application/json
    min-response-size: 1024

# JSF Configuration
jsf:
  primefaces:
    theme: nova-light
    font-awesome: true
  faces-servlet:
    url-pattern: "*.xhtml"
  
# Management endpoints
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
  metrics:
    export:
      prometheus:
        enabled: true

# Logging configuration
logging:
  level:
    com.eoffice.activitytracking: ${LOG_LEVEL:INFO}
    org.elasticsearch: WARN
    org.springframework.security: DEBUG
  pattern:
    console: "%d{yyyy-MM-dd HH:mm:ss} - %msg%n"
    file: "%d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%n"
  file:
    name: logs/activity-tracking.log
    max-size: 100MB
    max-history: 30

---
# Development profile
spring:
  config:
    activate:
      on-profile: development

# Elasticsearch configuration for development
elasticsearch:
  host: ${ELASTICSEARCH_HOST:localhost}
  port: ${ELASTICSEARCH_PORT:9200}
  username: ${ELASTICSEARCH_USERNAME:}
  password: ${ELASTICSEARCH_PASSWORD:}
  connection-timeout: 5000
  socket-timeout: 60000
  max-retry-timeout: 30000

# Log processing configuration
log-processing:
  enabled: true
  batch-size: 1000
  processing-interval: 30s
  log-sources:
    edcs:
      path: ${EDCS_LOG_PATH:/var/log/applications/edcs.log}
      enabled: true
    leav:
      path: ${LEAV_LOG_PATH:/var/log/applications/leav.log}
      enabled: true
    ofc-app:
      path: ${OFC_APP_LOG_PATH:/var/log/applications/ofc-app.log}
      enabled: true

# Security configuration
security:
  enabled: true
  cors:
    allowed-origins: "http://localhost:3000,http://localhost:8080"
    allowed-methods: "GET,POST,PUT,DELETE,OPTIONS"
    allowed-headers: "*"
    allow-credentials: true
  
# Cache configuration
cache:
  redis:
    host: ${REDIS_HOST:localhost}
    port: ${REDIS_PORT:6379}
    password: ${REDIS_PASSWORD:}
    timeout: 5000
    database: 0

---
# Production profile  
spring:
  config:
    activate:
      on-profile: production

# Production Elasticsearch configuration
elasticsearch:
  host: ${ELASTICSEARCH_HOST:elasticsearch-cluster.internal}
  port: ${ELASTICSEARCH_PORT:9200}
  username: ${ELASTICSEARCH_USERNAME}
  password: ${ELASTICSEARCH_PASSWORD}
  ssl:
    enabled: true
    trust-store: ${ELASTICSEARCH_TRUSTSTORE_PATH}
    trust-store-password: ${ELASTICSEARCH_TRUSTSTORE_PASSWORD}

# Production security
security:
  cors:
    allowed-origins: "https://activity-tracking.eoffice.local"
  ssl:
    enabled: true
    key-store: ${SSL_KEYSTORE_PATH}
    key-store-password: ${SSL_KEYSTORE_PASSWORD}

# Production logging
logging:
  level:
    com.eoffice.activitytracking: INFO
    org.springframework.security: WARN
  file:
    name: /var/log/activity-tracking/application.log
```

### Logback Configuration (logback-spring.xml)
```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    
    <!-- Console Appender -->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>
    
    <!-- File Appender -->
    <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>logs/activity-tracking.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>logs/activity-tracking-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <maxFileSize>100MB</maxFileSize>
            <maxHistory>30</maxHistory>
            <totalSizeCap>3GB</totalSizeCap>
        </rollingPolicy>
    </appender>
    
    <!-- Structured JSON Appender for production -->
    <springProfile name="production">
        <appender name="JSON" class="ch.qos.logback.core.rolling.RollingFileAppender">
            <file>/var/log/activity-tracking/application.json</file>
            <encoder class="net.logstash.logback.encoder.LoggingEventCompositeJsonEncoder">
                <providers>
                    <timestamp/>
                    <logLevel/>
                    <loggerName/>
                    <message/>
                    <mdc/>
                    <stackTrace/>
                </providers>
            </encoder>
            <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
                <fileNamePattern>/var/log/activity-tracking/application-%d{yyyy-MM-dd}.json</fileNamePattern>
                <maxHistory>30</maxHistory>
            </rollingPolicy>
        </appender>
    </springProfile>
    
    <!-- Logger configurations -->
    <logger name="com.eoffice.activitytracking" level="INFO" additivity="false">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <springProfile name="production">
            <appender-ref ref="JSON"/>
        </springProfile>
    </logger>
    
    <logger name="org.elasticsearch" level="WARN"/>
    <logger name="org.apache.http" level="WARN"/>
    <logger name="org.springframework.security" level="INFO"/>
    
    <root level="INFO">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="FILE"/>
        <springProfile name="production">
            <appender-ref ref="JSON"/>
        </springProfile>
    </root>
    
</configuration>
```

---

## 🔧 Common Development Tasks

### Building the Application
```bash
# Clean and compile
mvn clean compile

# Run tests
mvn test

# Generate test reports
mvn test jacoco:report

# Package application
mvn package

# Skip tests (for quick builds)
mvn package -DskipTests

# Install to local repository
mvn install
```

### Running the Application
```bash
# Run with Maven
mvn spring-boot:run

# Run with specific profile
mvn spring-boot:run -Dspring.profiles.active=development

# Run with custom properties
mvn spring-boot:run -Dspring.profiles.active=development -Dserver.port=8081

# Run packaged JAR
java -jar target/activity-tracking-system.war

# Run with JVM options
java -Xmx2g -Xms1g -jar target/activity-tracking-system.war
```

### Database Operations
```bash
# Check Elasticsearch health
curl -X GET "localhost:9200/_cluster/health?pretty"

# List all indices
curl -X GET "localhost:9200/_cat/indices?v"

# Create index template
curl -X PUT "localhost:9200/_index_template/activity-logs" \
  -H "Content-Type: application/json" \
  -d @elasticsearch/templates/activity-logs-template.json

# Search logs
curl -X GET "localhost:9200/activity-logs-*/_search?pretty" \
  -H "Content-Type: application/json" \
  -d '{
    "query": {
      "match": {
        "username": "sirat"
      }
    },
    "size": 10
  }'
```

### Testing Commands
```bash
# Run all tests
mvn test

# Run specific test class
mvn test -Dtest=LogAnalysisServiceTest

# Run specific test method
mvn test -Dtest=LogAnalysisServiceTest#testParseProductionLogs

# Run integration tests only
mvn test -Dtest=**/*IntegrationTest

# Run tests with coverage
mvn test jacoco:report

# Run performance tests
mvn test -Dtest=**/*PerformanceTest -Dspring.profiles.active=test
```

### Code Quality Commands
```bash
# Run SonarQube analysis
mvn sonar:sonar

# Check dependency updates
mvn versions:display-dependency-updates

# Check plugin updates
mvn versions:display-plugin-updates

# Format code (if using spotless plugin)
mvn spotless:apply

# Check code style
mvn checkstyle:check
```

---

## 🐛 Debugging and Troubleshooting

### Common Issues and Solutions

#### Issue 1: Elasticsearch Connection Failed
```bash
# Symptoms:
# - Application fails to start
# - Error: "Connection refused" or "No route to host"

# Solution:
# 1. Check if Elasticsearch is running
docker-compose ps elasticsearch

# 2. Check Elasticsearch logs
docker-compose logs elasticsearch

# 3. Verify Elasticsearch health
curl -X GET "localhost:9200/_cluster/health"

# 4. Restart Elasticsearch
docker-compose restart elasticsearch
```

#### Issue 2: JSF Pages Not Loading
```bash
# Symptoms:
# - 404 errors on .xhtml pages
# - White page with no content

# Solution:
# 1. Check web.xml configuration
# 2. Verify faces-config.xml
# 3. Check if JSF dependencies are in classpath
mvn dependency:tree | grep jsf

# 4. Clear browser cache and restart application
```

#### Issue 3: Log Processing Not Working
```bash
# Symptoms:
# - No data in dashboard
# - Logstash not processing logs

# Solution:
# 1. Check Logstash configuration
docker-compose logs logstash

# 2. Verify log file permissions
ls -la /var/log/applications/

# 3. Test log parsing patterns
# Use Grok Debugger in Kibana (http://localhost:5601)

# 4. Check Elasticsearch indices
curl -X GET "localhost:9200/_cat/indices?v"
```

### Debugging Configurations

#### Remote Debugging
```bash
# Add JVM options for remote debugging
export MAVEN_OPTS="-Xdebug -Xrunjdwp:transport=dt_socket,server=y,suspend=n,address=5005"
mvn spring-boot:run

# Or run with debug profile
java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=5005 \
     -jar target/activity-tracking-system.war
```

#### JVM Monitoring
```bash
# Monitor JVM with JConsole
jconsole

# Monitor with VisualVM
visualvm

# Check memory usage
jmap -histo <pid>

# Generate heap dump
jmap -dump:format=b,file=heapdump.hprof <pid>

# Monitor garbage collection
jstat -gc <pid> 5s
```

#### Application Monitoring
```bash
# Check application health
curl http://localhost:8080/activity-tracking/actuator/health

# View metrics
curl http://localhost:8080/activity-tracking/actuator/metrics

# View application info
curl http://localhost:8080/activity-tracking/actuator/info

# Monitor Prometheus metrics
curl http://localhost:8080/activity-tracking/actuator/prometheus
```

---

## 📝 Development Workflow

### Git Workflow
```bash
# Feature development workflow
git checkout develop
git pull origin develop
git checkout -b feature/log-analysis-enhancement

# Make changes...
git add .
git commit -m "feat: enhance log analysis with production patterns"

# Push feature branch
git push origin feature/log-analysis-enhancement

# Create pull request
# Merge after review
git checkout develop
git pull origin develop
git branch -d feature/log-analysis-enhancement
```

### Code Review Checklist
```markdown
## Code Review Checklist

### Functionality
- [ ] Code implements requirements correctly
- [ ] Edge cases are handled
- [ ] Error handling is appropriate
- [ ] Performance is acceptable

### Code Quality
- [ ] Code follows team standards
- [ ] Code is readable and well-commented
- [ ] No code duplication
- [ ] Proper logging is implemented

### Testing
- [ ] Unit tests are included
- [ ] Integration tests pass
- [ ] Code coverage is adequate
- [ ] Manual testing completed

### Security
- [ ] No security vulnerabilities
- [ ] Input validation implemented
- [ ] Authentication/authorization working
- [ ] Sensitive data is protected

### Documentation
- [ ] Code is self-documenting
- [ ] JavaDoc is complete
- [ ] README updated if needed
- [ ] API documentation updated
```

### Release Process
```bash
# Prepare release
git checkout develop
git pull origin develop
git checkout -b release/v1.0.0

# Update version numbers
mvn versions:set -DnewVersion=1.0.0

# Test release candidate
mvn clean package
# Run integration tests
# Run performance tests

# Merge to main
git checkout main
git merge release/v1.0.0
git tag v1.0.0
git push origin main --tags

# Merge back to develop
git checkout develop
git merge release/v1.0.0
git push origin develop

# Clean up
git branch -d release/v1.0.0
```

---

**Next:** [API Development Guide](./API-Development-Guide.md)  
**See Also:** [Testing Guide](./Testing-Guide.md) | [Deployment Guide](./Deployment-Guide.md)
