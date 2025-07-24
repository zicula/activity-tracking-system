# Deployment Guide - ACTIVITY-TRACKING System

## Document Information
**Project:** ACTIVITY-TRACKING System  
**Version:** 1.0  
**Date:** December 2024  
**Technology Stack:** Java JDK 1.8, JSF 2.x, Docker, Kubernetes, ELK Stack  
**Target Audience:** DevOps Engineers, System Administrators

---

## 🚀 Deployment Overview

### Deployment Strategy
```
Development → Staging → Pre-Production → Production
     ↓           ↓            ↓             ↓
   Docker    Kubernetes    Blue-Green    Rolling Update
```

### Environment Architecture
```
Production Environment
├── Load Balancer (HAProxy/Nginx)
├── Application Cluster (3 nodes)
│   ├── activity-tracking-app-1
│   ├── activity-tracking-app-2
│   └── activity-tracking-app-3
├── ELK Stack Cluster
│   ├── Elasticsearch Cluster (3 nodes)
│   ├── Logstash (2 nodes)
│   └── Kibana (1 node)
├── Redis Cluster (3 nodes)
└── Monitoring Stack
    ├── Prometheus
    ├── Grafana
    └── AlertManager
```

---

## 🐳 Docker Configuration

### Application Dockerfile
```dockerfile
# Multi-stage Dockerfile for production deployment
FROM openjdk:8-jdk-alpine AS build

# Install build dependencies
RUN apk add --no-cache curl maven

# Set working directory
WORKDIR /app

# Copy POM file and download dependencies
COPY pom.xml .
RUN mvn dependency:go-offline -B

# Copy source code and build
COPY src ./src
RUN mvn clean package -DskipTests

# Production stage
FROM openjdk:8-jre-alpine AS production

# Install runtime dependencies
RUN apk add --no-cache \
    curl \
    netcat-openbsd \
    bash \
    && addgroup -S appgroup \
    && adduser -S appuser -G appgroup

# Set working directory
WORKDIR /app

# Copy application JAR
COPY --from=build /app/target/activity-tracking-system.war app.war

# Copy configuration files
COPY docker/entrypoint.sh entrypoint.sh
COPY docker/health-check.sh health-check.sh

# Set permissions
RUN chmod +x entrypoint.sh health-check.sh \
    && chown -R appuser:appgroup /app

# Create directories for logs and temp files
RUN mkdir -p /app/logs /app/temp \
    && chown -R appuser:appgroup /app/logs /app/temp

# Switch to non-root user
USER appuser

# Expose application port
EXPOSE 8080

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
    CMD ./health-check.sh

# Set JVM options for production
ENV JAVA_OPTS="-Xms1g -Xmx2g -XX:+UseG1GC -XX:+UseStringDeduplication -Djava.security.egd=file:/dev/./urandom"

# Set Spring profiles
ENV SPRING_PROFILES_ACTIVE=production

# Entry point
ENTRYPOINT ["./entrypoint.sh"]
```

### Entrypoint Script
```bash
#!/bin/bash
# docker/entrypoint.sh

set -e

# Function to log messages
log() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1"
}

# Function to wait for dependencies
wait_for_dependency() {
    local host=$1
    local port=$2
    local service=$3
    local timeout=${4:-60}
    
    log "Waiting for $service at $host:$port..."
    
    for i in $(seq 1 $timeout); do
        if nc -z "$host" "$port"; then
            log "$service is available"
            return 0
        fi
        sleep 1
    done
    
    log "Timeout waiting for $service"
    exit 1
}

# Wait for required services
if [ -n "$ELASTICSEARCH_HOST" ]; then
    wait_for_dependency "$ELASTICSEARCH_HOST" "${ELASTICSEARCH_PORT:-9200}" "Elasticsearch"
fi

if [ -n "$REDIS_HOST" ]; then
    wait_for_dependency "$REDIS_HOST" "${REDIS_PORT:-6379}" "Redis"
fi

# Set JVM options based on environment
if [ "$ENVIRONMENT" = "production" ]; then
    export JAVA_OPTS="$JAVA_OPTS -server -XX:+UseG1GC -XX:MaxGCPauseMillis=200"
else
    export JAVA_OPTS="$JAVA_OPTS -XX:+UseParallelGC"
fi

# Start application
log "Starting Activity Tracking System..."
log "Active profile: $SPRING_PROFILES_ACTIVE"
log "JVM options: $JAVA_OPTS"

exec java $JAVA_OPTS -jar app.war "$@"
```

### Health Check Script
```bash
#!/bin/bash
# docker/health-check.sh

# Check application health endpoint
curl -f http://localhost:8080/activity-tracking/actuator/health || exit 1

# Check if application is responding
response=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:8080/activity-tracking/actuator/health)

if [ "$response" = "200" ]; then
    exit 0
else
    echo "Health check failed with status: $response"
    exit 1
fi
```

### Docker Compose for Development
```yaml
# docker-compose.yml
version: '3.8'

services:
  # Application
  activity-tracking-app:
    build:
      context: .
      dockerfile: Dockerfile
      target: production
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=development
      - ELASTICSEARCH_HOST=elasticsearch
      - ELASTICSEARCH_PORT=9200
      - REDIS_HOST=redis
      - REDIS_PORT=6379
      - DATABASE_URL=jdbc:postgresql://postgres:5432/activity_tracking
      - DATABASE_USERNAME=app_user
      - DATABASE_PASSWORD=app_password
    depends_on:
      - elasticsearch
      - redis
      - postgres
    networks:
      - activity-tracking-network
    volumes:
      - ./logs:/app/logs
      - ./config:/app/config:ro
    restart: unless-stopped

  # Elasticsearch
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
    container_name: activity-tracking-elasticsearch
    environment:
      - node.name=elasticsearch
      - cluster.name=activity-tracking-cluster
      - discovery.type=single-node
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms1g -Xmx1g"
      - xpack.security.enabled=false
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

  # Logstash
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

  # Kibana
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

  # Redis
  redis:
    image: redis:7-alpine
    container_name: activity-tracking-redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    networks:
      - activity-tracking-network
    command: redis-server --appendonly yes

  # PostgreSQL (for application metadata)
  postgres:
    image: postgres:13-alpine
    container_name: activity-tracking-postgres
    environment:
      POSTGRES_DB: activity_tracking
      POSTGRES_USER: app_user
      POSTGRES_PASSWORD: app_password
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./database/init:/docker-entrypoint-initdb.d:ro
    ports:
      - "5432:5432"
    networks:
      - activity-tracking-network

  # Nginx (Load Balancer)
  nginx:
    image: nginx:alpine
    container_name: activity-tracking-nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
      - ./nginx/ssl:/etc/nginx/ssl:ro
    networks:
      - activity-tracking-network
    depends_on:
      - activity-tracking-app

volumes:
  elasticsearch_data:
    driver: local
  redis_data:
    driver: local
  postgres_data:
    driver: local

networks:
  activity-tracking-network:
    driver: bridge
```

---

## ☸️ Kubernetes Deployment

### Namespace Configuration
```yaml
# k8s/namespace.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: activity-tracking
  labels:
    name: activity-tracking
    environment: production
```

### ConfigMap for Application Configuration
```yaml
# k8s/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: activity-tracking-config
  namespace: activity-tracking
data:
  application.yml: |
    spring:
      profiles:
        active: production
      
      datasource:
        url: ${DATABASE_URL}
        username: ${DATABASE_USERNAME}
        password: ${DATABASE_PASSWORD}
        hikari:
          maximum-pool-size: 20
          minimum-idle: 5
          connection-timeout: 30000
      
    elasticsearch:
      host: ${ELASTICSEARCH_HOST}
      port: ${ELASTICSEARCH_PORT}
      username: ${ELASTICSEARCH_USERNAME}
      password: ${ELASTICSEARCH_PASSWORD}
      connection-timeout: 5000
      socket-timeout: 60000
    
    cache:
      redis:
        host: ${REDIS_HOST}
        port: ${REDIS_PORT}
        password: ${REDIS_PASSWORD}
        timeout: 5000
    
    logging:
      level:
        com.eoffice.activitytracking: INFO
        org.springframework.security: WARN
      file:
        name: /app/logs/application.log
        max-size: 100MB
        max-history: 30
    
    management:
      endpoints:
        web:
          exposure:
            include: health,info,metrics,prometheus
      endpoint:
        health:
          show-details: when-authorized
  
  logback-spring.xml: |
    <?xml version="1.0" encoding="UTF-8"?>
    <configuration>
        <springProfile name="production">
            <appender name="FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
                <file>/app/logs/application.log</file>
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
                    <fileNamePattern>/app/logs/application-%d{yyyy-MM-dd}.log</fileNamePattern>
                    <maxHistory>30</maxHistory>
                </rollingPolicy>
            </appender>
            
            <root level="INFO">
                <appender-ref ref="FILE"/>
            </root>
        </springProfile>
    </configuration>
```

### Secret Configuration
```yaml
# k8s/secret.yaml
apiVersion: v1
kind: Secret
metadata:
  name: activity-tracking-secret
  namespace: activity-tracking
type: Opaque
data:
  database-url: <base64-encoded-database-url>
  database-username: <base64-encoded-username>
  database-password: <base64-encoded-password>
  elasticsearch-username: <base64-encoded-es-username>
  elasticsearch-password: <base64-encoded-es-password>
  redis-password: <base64-encoded-redis-password>
  jwt-secret: <base64-encoded-jwt-secret>
```

### Application Deployment
```yaml
# k8s/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: activity-tracking-app
  namespace: activity-tracking
  labels:
    app: activity-tracking
    tier: application
spec:
  replicas: 3
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: activity-tracking
      tier: application
  template:
    metadata:
      labels:
        app: activity-tracking
        tier: application
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/activity-tracking/actuator/prometheus"
    spec:
      serviceAccountName: activity-tracking-service-account
      
      # Init container to wait for dependencies
      initContainers:
      - name: wait-for-elasticsearch
        image: busybox:1.35
        command: ['sh', '-c']
        args:
        - |
          echo "Waiting for Elasticsearch..."
          until nc -z elasticsearch-service 9200; do
            echo "Elasticsearch is not ready yet..."
            sleep 5
          done
          echo "Elasticsearch is ready!"
      
      containers:
      - name: activity-tracking
        image: eoffice-registry.local/activity-tracking:latest
        imagePullPolicy: Always
        
        ports:
        - containerPort: 8080
          name: http
          protocol: TCP
        
        env:
        - name: SPRING_PROFILES_ACTIVE
          value: "production"
        - name: ELASTICSEARCH_HOST
          value: "elasticsearch-service"
        - name: ELASTICSEARCH_PORT
          value: "9200"
        - name: REDIS_HOST
          value: "redis-service"
        - name: REDIS_PORT
          value: "6379"
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: activity-tracking-secret
              key: database-url
        - name: DATABASE_USERNAME
          valueFrom:
            secretKeyRef:
              name: activity-tracking-secret
              key: database-username
        - name: DATABASE_PASSWORD
          valueFrom:
            secretKeyRef:
              name: activity-tracking-secret
              key: database-password
        - name: ELASTICSEARCH_USERNAME
          valueFrom:
            secretKeyRef:
              name: activity-tracking-secret
              key: elasticsearch-username
        - name: ELASTICSEARCH_PASSWORD
          valueFrom:
            secretKeyRef:
              name: activity-tracking-secret
              key: elasticsearch-password
        - name: REDIS_PASSWORD
          valueFrom:
            secretKeyRef:
              name: activity-tracking-secret
              key: redis-password
        
        # Resource limits
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        
        # Health checks
        livenessProbe:
          httpGet:
            path: /activity-tracking/actuator/health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
          timeoutSeconds: 10
          failureThreshold: 3
        
        readinessProbe:
          httpGet:
            path: /activity-tracking/actuator/health/readiness
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        
        # Volume mounts
        volumeMounts:
        - name: config-volume
          mountPath: /app/config
          readOnly: true
        - name: logs-volume
          mountPath: /app/logs
        - name: temp-volume
          mountPath: /app/temp
        
        # Security context
        securityContext:
          runAsNonRoot: true
          runAsUser: 1000
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: false
          capabilities:
            drop:
            - ALL
      
      volumes:
      - name: config-volume
        configMap:
          name: activity-tracking-config
      - name: logs-volume
        emptyDir: {}
      - name: temp-volume
        emptyDir: {}
      
      # Node selection
      nodeSelector:
        kubernetes.io/os: linux
      
      # Pod disruption budget
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - activity-tracking
              topologyKey: kubernetes.io/hostname
```

### Service Configuration
```yaml
# k8s/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: activity-tracking-service
  namespace: activity-tracking
  labels:
    app: activity-tracking
    tier: application
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "8080"
    prometheus.io/path: "/activity-tracking/actuator/prometheus"
spec:
  type: ClusterIP
  ports:
  - port: 8080
    targetPort: 8080
    protocol: TCP
    name: http
  selector:
    app: activity-tracking
    tier: application
  sessionAffinity: None
```

### Ingress Configuration
```yaml
# k8s/ingress.yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: activity-tracking-ingress
  namespace: activity-tracking
  annotations:
    kubernetes.io/ingress.class: "nginx"
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
    nginx.ingress.kubernetes.io/backend-protocol: "HTTP"
    nginx.ingress.kubernetes.io/proxy-body-size: "10m"
    nginx.ingress.kubernetes.io/proxy-connect-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-send-timeout: "300"
    nginx.ingress.kubernetes.io/proxy-read-timeout: "300"
    cert-manager.io/cluster-issuer: "letsencrypt-prod"
spec:
  tls:
  - hosts:
    - activity-tracking.eoffice.local
    secretName: activity-tracking-tls
  rules:
  - host: activity-tracking.eoffice.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: activity-tracking-service
            port:
              number: 8080
```

### Horizontal Pod Autoscaler
```yaml
# k8s/hpa.yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: activity-tracking-hpa
  namespace: activity-tracking
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: activity-tracking-app
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 10
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50
        periodSeconds: 60
```

---

## 🔧 ELK Stack Deployment

### Elasticsearch Cluster
```yaml
# k8s/elasticsearch.yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: elasticsearch
  namespace: activity-tracking
spec:
  serviceName: elasticsearch-service
  replicas: 3
  selector:
    matchLabels:
      app: elasticsearch
  template:
    metadata:
      labels:
        app: elasticsearch
    spec:
      containers:
      - name: elasticsearch
        image: docker.elastic.co/elasticsearch/elasticsearch:7.15.0
        ports:
        - containerPort: 9200
          name: http
        - containerPort: 9300
          name: transport
        env:
        - name: cluster.name
          value: "activity-tracking-cluster"
        - name: node.name
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: discovery.seed_hosts
          value: "elasticsearch-0.elasticsearch-service,elasticsearch-1.elasticsearch-service,elasticsearch-2.elasticsearch-service"
        - name: cluster.initial_master_nodes
          value: "elasticsearch-0,elasticsearch-1,elasticsearch-2"
        - name: ES_JAVA_OPTS
          value: "-Xms2g -Xmx2g"
        - name: xpack.security.enabled
          value: "true"
        - name: ELASTIC_PASSWORD
          valueFrom:
            secretKeyRef:
              name: activity-tracking-secret
              key: elasticsearch-password
        
        resources:
          requests:
            memory: "3Gi"
            cpu: "1000m"
          limits:
            memory: "4Gi"
            cpu: "2000m"
        
        volumeMounts:
        - name: elasticsearch-data
          mountPath: /usr/share/elasticsearch/data
        
        readinessProbe:
          httpGet:
            path: /_cluster/health
            port: 9200
            httpHeaders:
            - name: Authorization
              value: "Basic ZWxhc3RpYzpwYXNzd29yZA==" # elastic:password
          initialDelaySeconds: 30
          periodSeconds: 10
        
        livenessProbe:
          httpGet:
            path: /_cluster/health
            port: 9200
            httpHeaders:
            - name: Authorization
              value: "Basic ZWxhc3RpYzpwYXNzd29yZA=="
          initialDelaySeconds: 60
          periodSeconds: 30
  
  volumeClaimTemplates:
  - metadata:
      name: elasticsearch-data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: "fast-ssd"
      resources:
        requests:
          storage: 100Gi

---
apiVersion: v1
kind: Service
metadata:
  name: elasticsearch-service
  namespace: activity-tracking
spec:
  clusterIP: None
  selector:
    app: elasticsearch
  ports:
  - port: 9200
    name: http
  - port: 9300
    name: transport
```

### Logstash Deployment
```yaml
# k8s/logstash.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: logstash
  namespace: activity-tracking
spec:
  replicas: 2
  selector:
    matchLabels:
      app: logstash
  template:
    metadata:
      labels:
        app: logstash
    spec:
      containers:
      - name: logstash
        image: docker.elastic.co/logstash/logstash:7.15.0
        ports:
        - containerPort: 5044
          name: beats
        - containerPort: 5000
          name: tcp
        - containerPort: 5000
          protocol: UDP
          name: udp
        
        env:
        - name: LS_JAVA_OPTS
          value: "-Xmx1g -Xms1g"
        
        resources:
          requests:
            memory: "1.5Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        
        volumeMounts:
        - name: logstash-config
          mountPath: /usr/share/logstash/config/logstash.yml
          subPath: logstash.yml
        - name: logstash-pipeline
          mountPath: /usr/share/logstash/pipeline
        
        readinessProbe:
          httpGet:
            path: /
            port: 9600
          initialDelaySeconds: 30
          periodSeconds: 10
        
        livenessProbe:
          httpGet:
            path: /
            port: 9600
          initialDelaySeconds: 60
          periodSeconds: 30
      
      volumes:
      - name: logstash-config
        configMap:
          name: logstash-config
      - name: logstash-pipeline
        configMap:
          name: logstash-pipeline

---
apiVersion: v1
kind: Service
metadata:
  name: logstash-service
  namespace: activity-tracking
spec:
  selector:
    app: logstash
  ports:
  - port: 5044
    name: beats
  - port: 5000
    name: tcp
  - port: 5000
    protocol: UDP
    name: udp
```

### Kibana Deployment
```yaml
# k8s/kibana.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kibana
  namespace: activity-tracking
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kibana
  template:
    metadata:
      labels:
        app: kibana
    spec:
      containers:
      - name: kibana
        image: docker.elastic.co/kibana/kibana:7.15.0
        ports:
        - containerPort: 5601
          name: http
        
        env:
        - name: ELASTICSEARCH_HOSTS
          value: "http://elasticsearch-service:9200"
        - name: ELASTICSEARCH_USERNAME
          value: "elastic"
        - name: ELASTICSEARCH_PASSWORD
          valueFrom:
            secretKeyRef:
              name: activity-tracking-secret
              key: elasticsearch-password
        
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        
        readinessProbe:
          httpGet:
            path: /status
            port: 5601
          initialDelaySeconds: 60
          periodSeconds: 10
        
        livenessProbe:
          httpGet:
            path: /status
            port: 5601
          initialDelaySeconds: 120
          periodSeconds: 30

---
apiVersion: v1
kind: Service
metadata:
  name: kibana-service
  namespace: activity-tracking
spec:
  selector:
    app: kibana
  ports:
  - port: 5601
    name: http
```

---

## 📊 Monitoring and Observability

### Prometheus Configuration
```yaml
# k8s/prometheus-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: activity-tracking
data:
  prometheus.yml: |
    global:
      scrape_interval: 15s
      evaluation_interval: 15s
    
    rule_files:
      - "/etc/prometheus/rules/*.yml"
    
    scrape_configs:
    - job_name: 'activity-tracking-app'
      kubernetes_sd_configs:
      - role: endpoints
        namespaces:
          names:
          - activity-tracking
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
    
    - job_name: 'elasticsearch'
      static_configs:
      - targets: ['elasticsearch-service:9200']
      metrics_path: /_prometheus/metrics
    
    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
    
    alerting:
      alertmanagers:
      - static_configs:
        - targets:
          - alertmanager-service:9093

  rules.yml: |
    groups:
    - name: activity-tracking.rules
      rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.1
        for: 5m
        labels:
          severity: critical
        annotations:
          summary: "High error rate detected"
          description: "Error rate is {{ $value }} requests per second"
      
      - alert: HighResponseTime
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m])) > 2
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High response time detected"
          description: "95th percentile response time is {{ $value }} seconds"
      
      - alert: ElasticsearchDown
        expr: up{job="elasticsearch"} == 0
        for: 1m
        labels:
          severity: critical
        annotations:
          summary: "Elasticsearch is down"
          description: "Elasticsearch cluster is not responding"
      
      - alert: PodCrashLooping
        expr: rate(kube_pod_container_status_restarts_total[15m]) > 0
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "Pod is crash looping"
          description: "Pod {{ $labels.pod }} in namespace {{ $labels.namespace }} is crash looping"
```

### Grafana Dashboards
```json
{
  "dashboard": {
    "id": null,
    "title": "Activity Tracking System",
    "tags": ["activity-tracking", "java", "elasticsearch"],
    "timezone": "browser",
    "panels": [
      {
        "id": 1,
        "title": "Request Rate",
        "type": "graph",
        "targets": [
          {
            "expr": "rate(http_requests_total[1m])",
            "legendFormat": "{{method}} {{status}}"
          }
        ],
        "yAxes": [
          {
            "label": "Requests/sec"
          }
        ]
      },
      {
        "id": 2,
        "title": "Response Time",
        "type": "graph",
        "targets": [
          {
            "expr": "histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "95th percentile"
          },
          {
            "expr": "histogram_quantile(0.50, rate(http_request_duration_seconds_bucket[5m]))",
            "legendFormat": "50th percentile"
          }
        ]
      },
      {
        "id": 3,
        "title": "JVM Memory Usage",
        "type": "graph",
        "targets": [
          {
            "expr": "jvm_memory_used_bytes{area=\"heap\"}",
            "legendFormat": "Heap Used"
          },
          {
            "expr": "jvm_memory_max_bytes{area=\"heap\"}",
            "legendFormat": "Heap Max"
          }
        ]
      },
      {
        "id": 4,
        "title": "Elasticsearch Cluster Health",
        "type": "stat",
        "targets": [
          {
            "expr": "elasticsearch_cluster_health_status{cluster=\"activity-tracking-cluster\"}"
          }
        ]
      }
    ]
  }
}
```

---

## 🚀 Deployment Scripts

### Production Deployment Script
```bash
#!/bin/bash
# scripts/deploy-production.sh

set -e

# Configuration
NAMESPACE="activity-tracking"
IMAGE_TAG="${1:-latest}"
REGISTRY="eoffice-registry.local"
IMAGE_NAME="activity-tracking"
FULL_IMAGE="${REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m' # No Color

# Logging function
log() {
    echo -e "${GREEN}[$(date +'%Y-%m-%d %H:%M:%S')] $1${NC}"
}

warn() {
    echo -e "${YELLOW}[$(date +'%Y-%m-%d %H:%M:%S')] WARNING: $1${NC}"
}

error() {
    echo -e "${RED}[$(date +'%Y-%m-%d %H:%M:%S')] ERROR: $1${NC}"
    exit 1
}

# Check prerequisites
check_prerequisites() {
    log "Checking prerequisites..."
    
    # Check if kubectl is available
    if ! command -v kubectl &> /dev/null; then
        error "kubectl is not installed or not in PATH"
    fi
    
    # Check if kubectl can connect to cluster
    if ! kubectl cluster-info &> /dev/null; then
        error "Cannot connect to Kubernetes cluster"
    fi
    
    # Check if namespace exists
    if ! kubectl get namespace "$NAMESPACE" &> /dev/null; then
        log "Creating namespace $NAMESPACE..."
        kubectl create namespace "$NAMESPACE"
    fi
    
    log "Prerequisites check completed"
}

# Build and push Docker image
build_and_push_image() {
    log "Building Docker image..."
    
    # Build image
    docker build -t "$FULL_IMAGE" .
    
    if [ $? -ne 0 ]; then
        error "Docker build failed"
    fi
    
    log "Pushing image to registry..."
    docker push "$FULL_IMAGE"
    
    if [ $? -ne 0 ]; then
        error "Docker push failed"
    fi
    
    log "Image built and pushed successfully: $FULL_IMAGE"
}

# Deploy Elasticsearch cluster
deploy_elasticsearch() {
    log "Deploying Elasticsearch cluster..."
    
    kubectl apply -f k8s/elasticsearch.yaml -n "$NAMESPACE"
    
    # Wait for Elasticsearch to be ready
    log "Waiting for Elasticsearch cluster to be ready..."
    kubectl wait --for=condition=ready pod -l app=elasticsearch -n "$NAMESPACE" --timeout=300s
    
    log "Elasticsearch cluster deployed successfully"
}

# Deploy Logstash
deploy_logstash() {
    log "Deploying Logstash..."
    
    # Apply ConfigMaps first
    kubectl apply -f k8s/logstash-config.yaml -n "$NAMESPACE"
    kubectl apply -f k8s/logstash.yaml -n "$NAMESPACE"
    
    # Wait for Logstash to be ready
    kubectl wait --for=condition=available deployment/logstash -n "$NAMESPACE" --timeout=300s
    
    log "Logstash deployed successfully"
}

# Deploy Kibana
deploy_kibana() {
    log "Deploying Kibana..."
    
    kubectl apply -f k8s/kibana.yaml -n "$NAMESPACE"
    
    # Wait for Kibana to be ready
    kubectl wait --for=condition=available deployment/kibana -n "$NAMESPACE" --timeout=300s
    
    log "Kibana deployed successfully"
}

# Deploy application
deploy_application() {
    log "Deploying Activity Tracking application..."
    
    # Apply configurations
    kubectl apply -f k8s/configmap.yaml -n "$NAMESPACE"
    kubectl apply -f k8s/secret.yaml -n "$NAMESPACE"
    
    # Update image tag in deployment
    sed -i.bak "s|image: eoffice-registry.local/activity-tracking:.*|image: $FULL_IMAGE|g" k8s/deployment.yaml
    
    # Apply deployment
    kubectl apply -f k8s/deployment.yaml -n "$NAMESPACE"
    kubectl apply -f k8s/service.yaml -n "$NAMESPACE"
    kubectl apply -f k8s/ingress.yaml -n "$NAMESPACE"
    kubectl apply -f k8s/hpa.yaml -n "$NAMESPACE"
    
    # Wait for deployment to be ready
    log "Waiting for application deployment to be ready..."
    kubectl wait --for=condition=available deployment/activity-tracking-app -n "$NAMESPACE" --timeout=600s
    
    # Restore original deployment file
    mv k8s/deployment.yaml.bak k8s/deployment.yaml
    
    log "Application deployed successfully"
}

# Deploy monitoring
deploy_monitoring() {
    log "Deploying monitoring stack..."
    
    # Deploy Prometheus
    kubectl apply -f k8s/prometheus-config.yaml -n "$NAMESPACE"
    kubectl apply -f k8s/prometheus.yaml -n "$NAMESPACE"
    
    # Deploy Grafana
    kubectl apply -f k8s/grafana.yaml -n "$NAMESPACE"
    
    log "Monitoring stack deployed successfully"
}

# Verify deployment
verify_deployment() {
    log "Verifying deployment..."
    
    # Check pod status
    kubectl get pods -n "$NAMESPACE"
    
    # Check service endpoints
    kubectl get endpoints -n "$NAMESPACE"
    
    # Check ingress
    kubectl get ingress -n "$NAMESPACE"
    
    # Test application health
    log "Testing application health..."
    
    # Get service URL
    SERVICE_URL=$(kubectl get ingress activity-tracking-ingress -n "$NAMESPACE" -o jsonpath='{.spec.rules[0].host}')
    
    if [ -n "$SERVICE_URL" ]; then
        # Test health endpoint
        if curl -f "https://$SERVICE_URL/activity-tracking/actuator/health" > /dev/null 2>&1; then
            log "Application health check passed"
        else
            warn "Application health check failed - application may still be starting"
        fi
    else
        warn "Could not determine service URL for health check"
    fi
    
    log "Deployment verification completed"
}

# Rollback function
rollback_deployment() {
    log "Rolling back deployment..."
    
    kubectl rollout undo deployment/activity-tracking-app -n "$NAMESPACE"
    kubectl wait --for=condition=available deployment/activity-tracking-app -n "$NAMESPACE" --timeout=300s
    
    log "Rollback completed"
}

# Main deployment process
main() {
    log "Starting production deployment..."
    log "Image tag: $IMAGE_TAG"
    log "Target namespace: $NAMESPACE"
    
    # Confirmation for production deployment
    read -p "Are you sure you want to deploy to production? (yes/no): " confirm
    if [ "$confirm" != "yes" ]; then
        log "Deployment cancelled"
        exit 0
    fi
    
    # Execute deployment steps
    check_prerequisites
    build_and_push_image
    
    # Deploy ELK stack first (if not already deployed)
    if ! kubectl get statefulset elasticsearch -n "$NAMESPACE" &> /dev/null; then
        deploy_elasticsearch
    else
        log "Elasticsearch already deployed, skipping..."
    fi
    
    if ! kubectl get deployment logstash -n "$NAMESPACE" &> /dev/null; then
        deploy_logstash
    else
        log "Logstash already deployed, skipping..."
    fi
    
    if ! kubectl get deployment kibana -n "$NAMESPACE" &> /dev/null; then
        deploy_kibana
    else
        log "Kibana already deployed, skipping..."
    fi
    
    # Deploy application
    deploy_application
    
    # Deploy monitoring (if not already deployed)
    if ! kubectl get configmap prometheus-config -n "$NAMESPACE" &> /dev/null; then
        deploy_monitoring
    else
        log "Monitoring already deployed, skipping..."
    fi
    
    # Verify deployment
    verify_deployment
    
    log "Production deployment completed successfully!"
    log "Application URL: https://activity-tracking.eoffice.local"
    log "Kibana URL: https://kibana.eoffice.local"
    log "Grafana URL: https://grafana.eoffice.local"
}

# Handle script interruption
trap 'error "Deployment interrupted"' INT TERM

# Run main function
main "$@"
```

### Blue-Green Deployment Script
```bash
#!/bin/bash
# scripts/blue-green-deploy.sh

set -e

NAMESPACE="activity-tracking"
IMAGE_TAG="${1:-latest}"
CURRENT_COLOR="${2:-blue}"
TARGET_COLOR="${3:-green}"

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

# Deploy new version to target color
deploy_target() {
    log "Deploying $TARGET_COLOR environment with image tag $IMAGE_TAG"
    
    # Update deployment with new image
    kubectl set image deployment/activity-tracking-$TARGET_COLOR \
        activity-tracking=eoffice-registry.local/activity-tracking:$IMAGE_TAG \
        -n $NAMESPACE
    
    # Wait for deployment to complete
    kubectl rollout status deployment/activity-tracking-$TARGET_COLOR -n $NAMESPACE --timeout=600s
    
    log "$TARGET_COLOR deployment completed"
}

# Test target deployment
test_target() {
    log "Testing $TARGET_COLOR deployment"
    
    # Run health checks and basic tests
    TARGET_URL="http://activity-tracking-$TARGET_COLOR-service:8080"
    
    # Wait for pods to be ready
    kubectl wait --for=condition=ready pod -l app=activity-tracking,version=$TARGET_COLOR -n $NAMESPACE --timeout=300s
    
    # Run smoke tests
    kubectl run test-pod --rm -i --restart=Never --image=curlimages/curl:latest -- \
        curl -f "$TARGET_URL/activity-tracking/actuator/health" || {
        log "Health check failed for $TARGET_COLOR"
        return 1
    }
    
    log "$TARGET_COLOR deployment passed tests"
}

# Switch traffic to target
switch_traffic() {
    log "Switching traffic from $CURRENT_COLOR to $TARGET_COLOR"
    
    # Update service selector
    kubectl patch service activity-tracking-service -n $NAMESPACE \
        -p '{"spec":{"selector":{"version":"'$TARGET_COLOR'"}}}'
    
    log "Traffic switched to $TARGET_COLOR"
}

# Cleanup old deployment
cleanup_old() {
    log "Scaling down $CURRENT_COLOR deployment"
    
    kubectl scale deployment activity-tracking-$CURRENT_COLOR --replicas=0 -n $NAMESPACE
    
    log "Cleanup completed"
}

# Main blue-green deployment
main() {
    log "Starting blue-green deployment"
    log "Current: $CURRENT_COLOR, Target: $TARGET_COLOR, Image: $IMAGE_TAG"
    
    deploy_target
    test_target
    
    read -p "Switch traffic to $TARGET_COLOR? (yes/no): " confirm
    if [ "$confirm" = "yes" ]; then
        switch_traffic
        
        read -p "Cleanup $CURRENT_COLOR deployment? (yes/no): " cleanup
        if [ "$cleanup" = "yes" ]; then
            cleanup_old
        fi
        
        log "Blue-green deployment completed successfully"
    else
        log "Traffic switch cancelled"
    fi
}

main "$@"
```

---

## 🔐 Security Hardening

### Network Policies
```yaml
# k8s/network-policy.yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: activity-tracking-network-policy
  namespace: activity-tracking
spec:
  podSelector:
    matchLabels:
      app: activity-tracking
  policyTypes:
  - Ingress
  - Egress
  
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: ingress-nginx
    ports:
    - protocol: TCP
      port: 8080
  
  - from:
    - namespaceSelector:
        matchLabels:
          name: monitoring
    ports:
    - protocol: TCP
      port: 8080
  
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: elasticsearch
    ports:
    - protocol: TCP
      port: 9200
  
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - protocol: TCP
      port: 6379
  
  - to: [] # Allow DNS
    ports:
    - protocol: UDP
      port: 53
```

### Pod Security Policy
```yaml
# k8s/pod-security-policy.yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: activity-tracking-psp
  namespace: activity-tracking
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
    - ALL
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    - 'persistentVolumeClaim'
  runAsUser:
    rule: 'MustRunAsNonRoot'
  seLinux:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
```

---

## 📝 Maintenance and Operations

### Backup Scripts
```bash
#!/bin/bash
# scripts/backup-elasticsearch.sh

NAMESPACE="activity-tracking"
BACKUP_DIR="/backup/elasticsearch/$(date +%Y%m%d_%H%M%S)"
RETENTION_DAYS=30

log() {
    echo "[$(date +'%Y-%m-%d %H:%M:%S')] $1"
}

# Create Elasticsearch snapshot
create_snapshot() {
    log "Creating Elasticsearch snapshot..."
    
    # Create snapshot repository if not exists
    kubectl exec -n $NAMESPACE elasticsearch-0 -- \
        curl -X PUT "localhost:9200/_snapshot/backup" \
        -H "Content-Type: application/json" \
        -d '{
            "type": "fs",
            "settings": {
                "location": "/usr/share/elasticsearch/backup"
            }
        }'
    
    # Create snapshot
    SNAPSHOT_NAME="snapshot_$(date +%Y%m%d_%H%M%S)"
    kubectl exec -n $NAMESPACE elasticsearch-0 -- \
        curl -X PUT "localhost:9200/_snapshot/backup/$SNAPSHOT_NAME?wait_for_completion=true" \
        -H "Content-Type: application/json" \
        -d '{
            "indices": "activity-logs-*",
            "ignore_unavailable": true,
            "include_global_state": false
        }'
    
    log "Snapshot $SNAPSHOT_NAME created successfully"
}

# Cleanup old backups
cleanup_old_backups() {
    log "Cleaning up backups older than $RETENTION_DAYS days..."
    
    find /backup/elasticsearch -type d -mtime +$RETENTION_DAYS -exec rm -rf {} \;
    
    log "Cleanup completed"
}

main() {
    mkdir -p "$BACKUP_DIR"
    create_snapshot
    cleanup_old_backups
}

main "$@"
```

### Health Check Script
```bash
#!/bin/bash
# scripts/health-check.sh

NAMESPACE="activity-tracking"

check_pods() {
    echo "=== Pod Status ==="
    kubectl get pods -n $NAMESPACE
    
    # Check for crash looping pods
    CRASH_PODS=$(kubectl get pods -n $NAMESPACE --field-selector=status.phase=Failed -o name)
    if [ -n "$CRASH_PODS" ]; then
        echo "WARNING: Found failed pods: $CRASH_PODS"
    fi
}

check_services() {
    echo "=== Service Status ==="
    kubectl get services -n $NAMESPACE
    
    # Check endpoints
    kubectl get endpoints -n $NAMESPACE
}

check_application_health() {
    echo "=== Application Health ==="
    
    # Get application URL
    APP_URL=$(kubectl get ingress activity-tracking-ingress -n $NAMESPACE -o jsonpath='{.spec.rules[0].host}')
    
    if [ -n "$APP_URL" ]; then
        # Test health endpoint
        HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" "https://$APP_URL/activity-tracking/actuator/health")
        
        if [ "$HTTP_CODE" = "200" ]; then
            echo "✓ Application health check passed"
        else
            echo "✗ Application health check failed (HTTP $HTTP_CODE)"
        fi
    else
        echo "✗ Could not determine application URL"
    fi
}

check_elasticsearch() {
    echo "=== Elasticsearch Health ==="
    
    # Check cluster health
    kubectl exec -n $NAMESPACE elasticsearch-0 -- \
        curl -s "localhost:9200/_cluster/health?pretty"
}

main() {
    echo "Activity Tracking System Health Check"
    echo "====================================="
    echo "Timestamp: $(date)"
    echo
    
    check_pods
    echo
    check_services
    echo
    check_application_health
    echo
    check_elasticsearch
}

main "$@"
```

---

**Next:** [User Guide](./User-Guide.md)  
**See Also:** [Developer Setup Guide](./Developer-Setup-Guide.md) | [API Development Guide](./API-Development-Guide.md) | [Testing Guide](./Testing-Guide.md)
