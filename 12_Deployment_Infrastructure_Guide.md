# CleanStation Deployment & Infrastructure Guide

## Document Information
- **Document ID**: CLS-DOC-012
- **Version**: 1.0
- **Date**: December 2024
- **Project**: Torvan Medical CleanStation Production Workflow Digitalization

## Table of Contents
1. [Infrastructure Architecture](#infrastructure-architecture)
2. [Cloud Platform Strategy](#cloud-platform-strategy)
3. [Environment Management](#environment-management)
4. [CI/CD Pipeline](#cicd-pipeline)
5. [Monitoring & Logging](#monitoring--logging)
6. [Backup & Disaster Recovery](#backup--disaster-recovery)
7. [Performance Optimization](#performance-optimization)
8. [Cost Management](#cost-management)

## Infrastructure Architecture

### High-Level Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    Production Environment                    │
├─────────────────────────────────────────────────────────────┤
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐         │
│  │   CDN/WAF   │  │ Load Balancer│  │  Auto Scaling│         │
│  │  CloudFlare │  │   AWS ALB    │  │    Group     │         │
│  └─────────────┘  └─────────────┘  └─────────────┘         │
│                           │                                 │
│  ┌─────────────────────────────────────────────────────────┤
│  │              Application Tier                           │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │  │  Next.js    │  │  Next.js    │  │  Next.js    │    │
│  │  │ Container 1 │  │ Container 2 │  │ Container 3 │    │
│  │  │   ECS/EKS   │  │   ECS/EKS   │  │   ECS/EKS   │    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘    │
│  └─────────────────────────────────────────────────────────┤
│                           │                                 │
│  ┌─────────────────────────────────────────────────────────┤
│  │                Database Tier                            │
│  │  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐    │
│  │  │   Primary   │  │  Read Only  │  │    Redis    │    │
│  │  │ PostgreSQL  │  │  Replica    │  │   Cache     │    │
│  │  │    RDS      │  │    RDS      │  │  ElastiCache│    │
│  │  └─────────────┘  └─────────────┘  └─────────────┘    │
│  └─────────────────────────────────────────────────────────┤
└─────────────────────────────────────────────────────────────┘
```

### Container Architecture
```typescript
// Docker Configuration
interface ContainerSetup {
  nextjs: {
    image: "node:18-alpine"
    framework: "Next.js 14"
    buildTool: "Turbo"
    runtime: "Node.js 18"
  }
  
  nginx: {
    image: "nginx:alpine"
    purpose: "Reverse proxy & static assets"
    ssl: "Let's Encrypt certificates"
  }
  
  monitoring: {
    prometheus: "bitnami/prometheus"
    grafana: "grafana/grafana"
    alertmanager: "prom/alertmanager"
  }
}
```

## Cloud Platform Strategy

### AWS Services Architecture

#### Core Services
- **Compute**: ECS Fargate with Auto Scaling
- **Database**: RDS PostgreSQL Multi-AZ with Read Replicas
- **Cache**: ElastiCache Redis Cluster
- **Storage**: S3 with CloudFront CDN
- **Security**: IAM, Secrets Manager, Parameter Store
- **Networking**: VPC, ALB, Route 53

#### Service Configuration
```yaml
# ECS Service Definition
apiVersion: ecs/v1
kind: Service
metadata:
  name: cleanstation-app
spec:
  cluster: cleanstation-production
  taskDefinition: cleanstation-app:latest
  desiredCount: 3
  launchType: FARGATE
  
  networkConfiguration:
    awsvpcConfiguration:
      subnets:
        - subnet-private-1a
        - subnet-private-1b
        - subnet-private-1c
      securityGroups:
        - sg-app-tier
      assignPublicIp: DISABLED
  
  loadBalancers:
    - targetGroupArn: arn:aws:elasticloadbalancing:...
      containerName: cleanstation-app
      containerPort: 3000
  
  serviceRegistries:
    - registryArn: arn:aws:servicediscovery:...
```

#### Database Configuration
```sql
-- RDS PostgreSQL Configuration
-- Instance: db.r6g.xlarge (4 vCPU, 32 GB RAM)
-- Storage: 500 GB GP3 SSD with 3000 IOPS
-- Backup: 30-day retention, automated backups
-- Multi-AZ: Enabled for high availability

CREATE DATABASE cleanstation_prod
  WITH ENCODING 'UTF8'
  LC_COLLATE = 'en_US.UTF-8'
  LC_CTYPE = 'en_US.UTF-8'
  TEMPLATE template0;

-- Connection pooling configuration
-- PgBouncer settings:
-- max_client_conn = 100
-- default_pool_size = 25
-- pool_mode = transaction
```

## Environment Management

### Environment Strategy
```typescript
interface Environments {
  development: {
    purpose: "Local development"
    infrastructure: "Docker Compose"
    database: "PostgreSQL local instance"
    domains: ["localhost:3000"]
  }
  
  staging: {
    purpose: "Integration testing & QA"
    infrastructure: "AWS ECS (single instance)"
    database: "RDS PostgreSQL (db.t3.medium)"
    domains: ["staging.cleanstation.torvan.com"]
    monitoring: "Basic CloudWatch"
  }
  
  production: {
    purpose: "Live manufacturing operations"
    infrastructure: "AWS ECS (multi-AZ, auto-scaling)"
    database: "RDS PostgreSQL Multi-AZ (db.r6g.xlarge)"
    domains: ["cleanstation.torvan.com"]
    monitoring: "Full observability stack"
    compliance: "ISO 13485:2016"
  }
}
```

### Environment Variables Management
```bash
# Production Environment Variables (AWS Systems Manager)
/cleanstation/prod/database/url          # RDS connection string
/cleanstation/prod/database/readonly_url # Read replica connection
/cleanstation/prod/jwt/private_key       # RSA private key for JWT
/cleanstation/prod/jwt/public_key        # RSA public key for JWT
/cleanstation/prod/encryption/key        # AES-256 encryption key
/cleanstation/prod/smtp/credentials      # Email service credentials
/cleanstation/prod/s3/credentials        # S3 access credentials
/cleanstation/prod/redis/url             # ElastiCache connection string
```

## CI/CD Pipeline

### GitHub Actions Workflow
```yaml
name: CleanStation CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:15
        env:
          POSTGRES_PASSWORD: postgres
          POSTGRES_DB: cleanstation_test
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linting
        run: npm run lint
      
      - name: Run type checking
        run: npm run type-check
      
      - name: Run unit tests
        run: npm run test:unit
        env:
          DATABASE_URL: postgresql://postgres:postgres@localhost:5432/cleanstation_test
      
      - name: Run integration tests
        run: npm run test:integration
      
      - name: Run E2E tests
        run: npm run test:e2e
      
      - name: Build application
        run: npm run build
      
      - name: Security audit
        run: npm audit --audit-level=high

  deploy-staging:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/develop'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Build and push Docker image
        run: |
          aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY
          docker build -t $ECR_REGISTRY/cleanstation:staging-$GITHUB_SHA .
          docker push $ECR_REGISTRY/cleanstation:staging-$GITHUB_SHA
      
      - name: Deploy to staging
        run: |
          aws ecs update-service \
            --cluster cleanstation-staging \
            --service cleanstation-app \
            --task-definition cleanstation-app:staging-$GITHUB_SHA \
            --force-new-deployment

  deploy-production:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: production
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Configure AWS credentials
        uses: aws-actions/configure-aws-credentials@v4
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      
      - name: Build and push Docker image
        run: |
          aws ecr get-login-password | docker login --username AWS --password-stdin $ECR_REGISTRY
          docker build -t $ECR_REGISTRY/cleanstation:prod-$GITHUB_SHA .
          docker push $ECR_REGISTRY/cleanstation:prod-$GITHUB_SHA
      
      - name: Database migrations
        run: |
          npm run db:migrate:prod
      
      - name: Deploy to production (Blue-Green)
        run: |
          # Deploy to green environment
          aws ecs update-service \
            --cluster cleanstation-production \
            --service cleanstation-app-green \
            --task-definition cleanstation-app:prod-$GITHUB_SHA
          
          # Health check
          ./scripts/health-check.sh cleanstation-green.internal
          
          # Switch traffic
          aws elbv2 modify-rule \
            --rule-arn $TARGET_GROUP_RULE_ARN \
            --actions Type=forward,TargetGroupArn=$GREEN_TARGET_GROUP_ARN
          
          # Update blue environment for next deployment
          sleep 300  # Wait for traffic to drain
          aws ecs update-service \
            --cluster cleanstation-production \
            --service cleanstation-app-blue \
            --task-definition cleanstation-app:prod-$GITHUB_SHA
```

### Deployment Scripts
```bash
#!/bin/bash
# scripts/deploy.sh

set -e

ENVIRONMENT=${1:-staging}
IMAGE_TAG=${2:-latest}

echo "Deploying CleanStation to $ENVIRONMENT environment..."

# Validate environment
if [[ ! "$ENVIRONMENT" =~ ^(staging|production)$ ]]; then
  echo "Error: Environment must be 'staging' or 'production'"
  exit 1
fi

# Build and tag Docker image
docker build -t cleanstation:$IMAGE_TAG .
docker tag cleanstation:$IMAGE_TAG $ECR_REGISTRY/cleanstation:$ENVIRONMENT-$IMAGE_TAG

# Push to ECR
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin $ECR_REGISTRY
docker push $ECR_REGISTRY/cleanstation:$ENVIRONMENT-$IMAGE_TAG

# Update ECS service
aws ecs update-service \
  --cluster cleanstation-$ENVIRONMENT \
  --service cleanstation-app \
  --force-new-deployment

# Wait for deployment
aws ecs wait services-stable \
  --cluster cleanstation-$ENVIRONMENT \
  --services cleanstation-app

echo "Deployment to $ENVIRONMENT completed successfully!"
```

## Monitoring & Logging

### Observability Stack
```typescript
interface MonitoringStack {
  metrics: {
    prometheus: "Metrics collection"
    grafana: "Visualization and dashboards"
    cloudwatch: "AWS native metrics"
  }
  
  logging: {
    winston: "Application logging"
    cloudwatchLogs: "Centralized log storage"
    elasticsearch: "Log search and analysis"
  }
  
  tracing: {
    jaeger: "Distributed tracing"
    xray: "AWS X-Ray integration"
  }
  
  alerting: {
    alertmanager: "Alert routing"
    pagerduty: "Incident management"
    slack: "Team notifications"
  }
}
```

### Key Metrics & Alerts
```yaml
# Grafana Dashboard Configuration
dashboard:
  title: "CleanStation Production Metrics"
  
  panels:
    - title: "Application Health"
      metrics:
        - http_requests_total
        - http_request_duration_seconds
        - nodejs_heap_used_bytes
        - process_cpu_usage_percent
    
    - title: "Database Performance"
      metrics:
        - postgresql_connections_active
        - postgresql_query_duration_seconds
        - postgresql_deadlocks_total
        - postgresql_cache_hit_ratio
    
    - title: "Business Metrics"
      metrics:
        - orders_created_total
        - qc_checks_completed_total
        - assembly_tasks_completed_total
        - user_sessions_active

alerts:
  - name: "High Error Rate"
    condition: "rate(http_requests_total{status=~'5..'}[5m]) > 0.05"
    severity: "critical"
    notification: "pagerduty"
  
  - name: "Database Connection Pool Exhausted"
    condition: "postgresql_connections_active > 80"
    severity: "warning"
    notification: "slack"
  
  - name: "Memory Usage High"
    condition: "nodejs_heap_used_bytes / nodejs_heap_size_bytes > 0.9"
    severity: "warning"
    notification: "slack"
```

### Application Logging
```typescript
// logging/logger.ts
import winston from 'winston';
import CloudWatchTransport from 'winston-cloudwatch';

const logger = winston.createLogger({
  level: process.env.LOG_LEVEL || 'info',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  defaultMeta: {
    service: 'cleanstation',
    environment: process.env.NODE_ENV,
    version: process.env.APP_VERSION
  },
  transports: [
    // Console transport for development
    new winston.transports.Console({
      format: winston.format.combine(
        winston.format.colorize(),
        winston.format.simple()
      )
    }),
    
    // CloudWatch transport for production
    new CloudWatchTransport({
      logGroupName: '/aws/ecs/cleanstation',
      logStreamName: `${process.env.HOSTNAME}-${new Date().toISOString().split('T')[0]}`,
      awsRegion: 'us-east-1',
      jsonMessage: true
    })
  ]
});

// Audit logging for compliance
export const auditLogger = winston.createLogger({
  level: 'info',
  format: winston.format.json(),
  transports: [
    new winston.transports.File({
      filename: '/var/log/cleanstation/audit.log',
      maxsize: 100000000, // 100MB
      maxFiles: 10
    }),
    new CloudWatchTransport({
      logGroupName: '/aws/ecs/cleanstation/audit',
      logStreamName: 'audit-log'
    })
  ]
});

export default logger;
```

## Backup & Disaster Recovery

### Backup Strategy
```typescript
interface BackupStrategy {
  database: {
    automated: {
      frequency: "Daily at 2:00 AM UTC"
      retention: "30 days"
      method: "RDS automated backups"
    }
    
    manual: {
      frequency: "Before major deployments"
      retention: "90 days"
      method: "RDS manual snapshots"
    }
    
    pointInTime: {
      enabled: true
      retention: "7 days"
      method: "RDS point-in-time recovery"
    }
  }
  
  files: {
    s3: {
      versioning: "Enabled"
      crossRegionReplication: "us-west-2"
      lifecyclePolicy: "Transition to IA after 30 days, Glacier after 90 days"
    }
  }
  
  configuration: {
    infrastructure: "Terraform state files in S3 with versioning"
    secrets: "AWS Secrets Manager with automatic rotation"
    environment: "Parameter Store with versioning"
  }
}
```

### Disaster Recovery Plan
```yaml
# RTO: 4 hours, RPO: 1 hour
disaster_recovery:
  scenarios:
    - name: "Regional AWS Outage"
      impact: "Complete service unavailability"
      response:
        - Switch DNS to backup region (us-west-2)
        - Restore RDS from cross-region replica
        - Deploy application to backup region
        - Validate functionality and data integrity
      
    - name: "Database Corruption"
      impact: "Data loss potential"
      response:
        - Stop application traffic
        - Restore from latest automated backup
        - Apply transaction logs for point-in-time recovery
        - Validate data integrity
        - Resume operations
    
    - name: "Security Incident"
      impact: "Potential data breach"
      response:
        - Isolate affected systems
        - Rotate all credentials and certificates
        - Audit access logs
        - Notify stakeholders per compliance requirements
        - Implement additional security measures

  testing:
    frequency: "Quarterly"
    scenarios: ["Backup restoration", "Failover procedures", "Security incident"]
    documentation: "Test results logged in incident management system"
```

## Performance Optimization

### Application Performance
```typescript
// Performance optimization configurations
interface PerformanceConfig {
  nextjs: {
    output: "standalone"
    experimental: {
      serverComponentsExternalPackages: ["@prisma/client"]
    }
    compiler: {
      removeConsole: true // Production only
    }
  }
  
  database: {
    connectionPooling: {
      maxConnections: 100
      idleTimeout: 30000
      connectionTimeout: 60000
    }
    
    indexing: {
      strategy: "Analyze query patterns and create targeted indexes"
      maintenance: "REINDEX and ANALYZE scheduled weekly"
    }
    
    queryOptimization: {
      enableQueryPlan: true
      slowQueryLogging: "Queries > 1000ms"
      explainAnalyze: "Enabled for development"
    }
  }
  
  caching: {
    redis: {
      maxMemory: "2gb"
      evictionPolicy: "allkeys-lru"
      persistenceEnabled: false
    }
    
    cdn: {
      provider: "CloudFront"
      cachingStrategy: "Aggressive for static assets"
      edgeLocations: "Global distribution"
    }
  }
}
```

### Load Testing Configuration
```javascript
// k6 Load Testing Script
import http from 'k6/http';
import { check, sleep } from 'k6';

export let options = {
  stages: [
    { duration: '2m', target: 100 }, // Ramp up to 100 users
    { duration: '5m', target: 100 }, // Stay at 100 users
    { duration: '2m', target: 200 }, // Ramp up to 200 users
    { duration: '5m', target: 200 }, // Stay at 200 users
    { duration: '2m', target: 0 },   // Ramp down to 0 users
  ],
  thresholds: {
    http_req_duration: ['p(99)<1500'], // 99% of requests must complete below 1.5s
    http_req_failed: ['rate<0.1'],     // Error rate must be below 10%
  },
};

export default function () {
  let response = http.get('https://cleanstation.torvan.com/api/health');
  check(response, {
    'status is 200': (r) => r.status === 200,
    'response time < 500ms': (r) => r.timings.duration < 500,
  });
  
  // Simulate user workflow
  http.post('https://cleanstation.torvan.com/api/auth/login', {
    email: 'test.user@torvan.com',
    password: 'TestPassword123!'
  });
  
  sleep(1);
}
```

## Cost Management

### Cost Optimization Strategy
```typescript
interface CostOptimization {
  compute: {
    rightSizing: "Monitor CPU/memory utilization and adjust instance types"
    autoScaling: "Scale down during off-hours (nights/weekends)"
    spotInstances: "Use for non-critical workloads and testing"
    reservedInstances: "1-year reservations for predictable workloads"
  }
  
  storage: {
    s3Lifecycle: "Transition to IA/Glacier based on access patterns"
    ebsOptimization: "GP3 volumes with provisioned IOPS only when needed"
    cleanupPolicy: "Automated deletion of old logs and temporary files"
  }
  
  networking: {
    dataTransfer: "Minimize cross-AZ data transfer"
    cloudFront: "Reduce origin requests through aggressive caching"
    natGateway: "Single NAT Gateway per region for development environments"
  }
  
  monitoring: {
    costAlerts: "Budget alerts at 80% and 100% of monthly allocation"
    unusedResources: "Weekly automated reports on unused resources"
    rightsizingRecommendations: "AWS Trusted Advisor integration"
  }
}
```

### Monthly Cost Estimates
```yaml
# Production Environment (Monthly Estimates)
estimated_costs:
  compute:
    ecs_fargate: "$420/month (3 tasks * $0.04/hour * 24 * 30)"
    load_balancer: "$16/month (1 ALB)"
    nat_gateway: "$32/month (1 NAT Gateway)"
  
  database:
    rds_primary: "$580/month (db.r6g.xlarge)"
    rds_replica: "$290/month (db.r6g.large)"
    backup_storage: "$50/month (500GB * 30 days)"
  
  storage:
    s3_storage: "$23/month (1TB)"
    cloudfront: "$50/month (data transfer)"
    ecr: "$10/month (container images)"
  
  monitoring:
    cloudwatch: "$30/month (metrics and logs)"
    
  total_estimated: "$1,501/month"
  
  cost_optimization_targets:
    - "Reserved instances: Save 30-40% on compute costs"
    - "Auto-scaling: Reduce off-hours costs by 50%"
    - "Storage lifecycle: Save 60% on long-term storage"
```

## Implementation Checklist

### Pre-Deployment
- [ ] AWS account setup and IAM roles configured
- [ ] Domain and SSL certificates provisioned
- [ ] Environment variables and secrets configured
- [ ] Database schemas and initial data loaded
- [ ] CI/CD pipeline tested and validated
- [ ] Security scanning and penetration testing completed
- [ ] Load testing and performance validation
- [ ] Backup and recovery procedures tested
- [ ] Monitoring and alerting configured
- [ ] Disaster recovery plan documented and tested

### Post-Deployment
- [ ] Health checks and monitoring validated
- [ ] User acceptance testing completed
- [ ] Staff training conducted
- [ ] Documentation updated
- [ ] Compliance audit completed
- [ ] Performance baseline established
- [ ] Cost monitoring activated
- [ ] Incident response procedures tested

---

**Document Version**: 1.0  
**Last Updated**: December 2024  
**Next Review**: March 2025 