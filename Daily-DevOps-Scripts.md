#!/bin/bash
# Collection of Daily DevOps Bash Scripts

# 1. Quick Docker Cleanup
```
docker_cleanup() {
    echo "🧹 Cleaning up Docker resources..."
    docker system prune -f
    docker volume prune -f
    docker image prune -a -f
}
```
# 2. AWS Cost Check
```
aws_cost_check() {
    aws ce get-cost-and-usage \
        --time-period Start=2024-01-01,End=2024-01-31 \
        --granularity MONTHLY \
        --metrics BlendedCost \
        --query 'ResultsByTime[0].Total.BlendedCost.Amount' \
        --output text
}
```
# 3. Kubernetes Pod Restart
```
k8s_restart_deployment() {
    local deployment=$1
    local namespace=${2:-default}
    kubectl rollout restart deployment/$deployment -n $namespace
    kubectl rollout status deployment/$deployment -n $namespace
}
```
# 4. SSL Certificate Expiry Check
```
check_ssl_expiry() {
    local domain=$1
    local days=$(echo | openssl s_client -servername $domain -connect $domain:443 2>/dev/null | \
                openssl x509 -noout -checkend $((30*24*3600)) && echo "OK" || echo "EXPIRING")
    echo "$domain: $days"
}
```
# 5. Log Tail with Filtering
```
tail_app_logs() {
    local service=$1
    local level=${2:-ERROR}
    journalctl -u $service -f | grep $level
}
```
# 6. Database Connection Test
```
test_db_connection() {
    local host=$1
    local port=${2:-3306}
    timeout 5 bash -c "</dev/tcp/$host/$port" && echo "✅ Connected" || echo "❌ Failed"
}
```
# 7. Git Branch Cleanup
```
git_cleanup_branches() {
    git branch --merged | grep -v "\*\|main\|master" | xargs -n 1 git branch -d
    git remote prune origin
}
```
# 8. System Resource Check
```
system_check() {
    echo "CPU: $(top -bn1 | grep "Cpu(s)" | awk '{print $2}' | cut -d'%' -f1)%"
    echo "Memory: $(free | awk 'NR==2{printf "%.1f%%", $3*100/$2}')"
    echo "Disk: $(df -h / | awk 'NR==2{print $5}')"
}
```
# 9. Service Health Check
```
health_check() {
    local url=$1
    local status=$(curl -s -o /dev/null -w "%{http_code}" $url)
    [ $status -eq 200 ] && echo "✅ $url" || echo "❌ $url ($status)"
}
```
# 10. Backup Verification
```
verify_backup() {
    local backup_path=$1
    local max_age_hours=${2:-24}
    
    if [ -f "$backup_path" ]; then
        local age=$(( ($(date +%s) - $(stat -c %Y "$backup_path")) / 3600 ))
        [ $age -lt $max_age_hours ] && echo "✅ Backup OK" || echo "⚠️  Backup old ($age hours)"
    else
        echo "❌ Backup missing"
    fi
}
```
# Usage examples:
# docker_cleanup
# k8s_restart_deployment myapp production
# check_ssl_expiry example.com
# health_check https://api.example.com/health

-----------------------------------------------------------------------------------------
# Real-World Bash Scripting Uses in DevOps

## 1. CI/CD Pipeline Scripts

### Build and Deploy Script
```
#!/bin/bash
# build-deploy.sh
set -e

APP_NAME="myapp"
VERSION=$(git rev-parse --short HEAD)
REGISTRY="123456789.dkr.ecr.us-east-1.amazonaws.com"

echo "Building $APP_NAME:$VERSION"
docker build -t $APP_NAME:$VERSION .
docker tag $APP_NAME:$VERSION $REGISTRY/$APP_NAME:$VERSION

aws ecr get-login-password | docker login --username AWS --password-stdin $REGISTRY
docker push $REGISTRY/$APP_NAME:$VERSION

kubectl set image deployment/$APP_NAME $APP_NAME=$REGISTRY/$APP_NAME:$VERSION
```

### Test Automation Script
```
#!/bin/bash
# run-tests.sh
export NODE_ENV=test

npm install
npm run lint
npm run test:unit
npm run test:integration

if [ $? -eq 0 ]; then
    echo "✅ All tests passed"
    exit 0
else
    echo "❌ Tests failed"
    exit 1
fi
```

## 2. Infrastructure Management

### AWS Resource Cleanup
```
#!/bin/bash
# cleanup-aws-resources.sh
REGION="us-east-1"
TAG_KEY="Environment"
TAG_VALUE="dev"

# Stop and terminate EC2 instances
aws ec2 describe-instances \
  --filters "Name=tag:$TAG_KEY,Values=$TAG_VALUE" "Name=instance-state-name,Values=running" \
  --query 'Reservations[].Instances[].InstanceId' --output text | \
xargs -r aws ec2 terminate-instances --instance-ids

# Delete unused EBS volumes
aws ec2 describe-volumes \
  --filters "Name=status,Values=available" \
  --query 'Volumes[].VolumeId' --output text | \
xargs -r aws ec2 delete-volume --volume-id
```

### Kubernetes Health Check
```
#!/bin/bash
# k8s-health-check.sh
NAMESPACE="production"

kubectl get pods -n $NAMESPACE --field-selector=status.phase!=Running -o json | \
jq -r '.items[] | "\(.metadata.name) is \(.status.phase)"'

UNHEALTHY=$(kubectl get pods -n $NAMESPACE --field-selector=status.phase!=Running --no-headers | wc -l)

if [ $UNHEALTHY -gt 0 ]; then
    echo "⚠️  $UNHEALTHY unhealthy pods found"
    exit 1
fi
```

## 3. Monitoring and Alerting

### System Health Monitor
```
#!/bin/bash
# system-monitor.sh
THRESHOLD=80
DISK_USAGE=$(df / | awk 'NR==2 {print $5}' | sed 's/%//')
MEMORY_USAGE=$(free | awk 'NR==2{printf "%.0f", $3*100/$2}')

if [ $DISK_USAGE -gt $THRESHOLD ]; then
    aws sns publish --topic-arn arn:aws:sns:us-east-1:123456789:alerts \
      --message "Disk usage is ${DISK_USAGE}% on $(hostname)"
fi

if [ $MEMORY_USAGE -gt $THRESHOLD ]; then
    aws sns publish --topic-arn arn:aws:sns:us-east-1:123456789:alerts \
      --message "Memory usage is ${MEMORY_USAGE}% on $(hostname)"
fi
```

### Log Analysis Script
```
#!/bin/bash
# analyze-logs.sh
LOG_FILE="/var/log/application.log"
ERROR_COUNT=$(grep -c "ERROR" $LOG_FILE)
WARN_COUNT=$(grep -c "WARN" $LOG_FILE)

echo "Errors: $ERROR_COUNT, Warnings: $WARN_COUNT"

if [ $ERROR_COUNT -gt 10 ]; then
    grep "ERROR" $LOG_FILE | tail -5 | \
    aws sns publish --topic-arn arn:aws:sns:us-east-1:123456789:alerts \
      --message "High error count: $ERROR_COUNT errors found"
fi
```

## 4. Database Operations

### Database Backup Script
```
#!/bin/bash
# db-backup.sh
DB_HOST="prod-db.cluster-xyz.us-east-1.rds.amazonaws.com"
DB_NAME="myapp"
BACKUP_DIR="/backups"
DATE=$(date +%Y%m%d_%H%M%S)

mysqldump -h $DB_HOST -u $DB_USER -p$DB_PASS $DB_NAME > $BACKUP_DIR/${DB_NAME}_${DATE}.sql

aws s3 cp $BACKUP_DIR/${DB_NAME}_${DATE}.sql s3://my-backups/database/

# Keep only last 7 days of local backups
find $BACKUP_DIR -name "*.sql" -mtime +7 -delete
```

### Database Migration Script
```
#!/bin/bash
# migrate-db.sh
ENVIRONMENT=${1:-staging}
CONFIG_FILE="config/${ENVIRONMENT}.env"

source $CONFIG_FILE

echo "Running migrations for $ENVIRONMENT"
npm run migrate:up

if [ $? -eq 0 ]; then
    echo "✅ Migration completed successfully"
else
    echo "❌ Migration failed, rolling back"
    npm run migrate:down
    exit 1
fi
```

## 5. Security and Compliance

### Security Scan Script
```
#!/bin/bash
# security-scan.sh
IMAGE_NAME=$1

echo "Scanning $IMAGE_NAME for vulnerabilities"
trivy image --severity HIGH,CRITICAL $IMAGE_NAME

VULN_COUNT=$(trivy image --format json $IMAGE_NAME | jq '.Results[].Vulnerabilities | length')

if [ $VULN_COUNT -gt 0 ]; then
    echo "❌ Found $VULN_COUNT vulnerabilities"
    exit 1
fi
```

### SSL Certificate Check
```
#!/bin/bash
# check-ssl-certs.sh
DOMAINS=("example.com" "api.example.com" "admin.example.com")

for domain in "${DOMAINS[@]}"; do
    expiry=$(echo | openssl s_client -servername $domain -connect $domain:443 2>/dev/null | \
             openssl x509 -noout -dates | grep notAfter | cut -d= -f2)
    
    expiry_epoch=$(date -d "$expiry" +%s)
    current_epoch=$(date +%s)
    days_left=$(( (expiry_epoch - current_epoch) / 86400 ))
    
    if [ $days_left -lt 30 ]; then
        echo "⚠️  SSL certificate for $domain expires in $days_left days"
    fi
done
```

## 6. Development Workflow

### Git Hook Script
```
#!/bin/bash
# pre-commit hook
npm run lint
npm run test

if [ $? -ne 0 ]; then
    echo "❌ Pre-commit checks failed"
    exit 1
fi

# Check for secrets
if git diff --cached --name-only | xargs grep -l "password\|secret\|key" > /dev/null; then
    echo "❌ Potential secrets detected"
    exit 1
fi
```

### Environment Setup Script
```
#!/bin/bash
# setup-dev-env.sh
echo "Setting up development environment..."

# Install dependencies
if command -v brew &> /dev/null; then
    brew install node docker kubectl terraform
elif command -v apt-get &> /dev/null; then
    curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
    sudo apt-get install -y nodejs docker.io
fi

# Configure tools
kubectl config use-context dev-cluster
terraform workspace select dev

echo "✅ Development environment ready"
```

## 7. Performance and Optimization

### Performance Test Script
```
#!/bin/bash
# performance-test.sh
URL="https://api.example.com/health"
REQUESTS=1000
CONCURRENCY=10

ab -n $REQUESTS -c $CONCURRENCY $URL > /tmp/perf_results.txt

AVG_TIME=$(grep "Time per request" /tmp/perf_results.txt | head -1 | awk '{print $4}')
FAILED_REQUESTS=$(grep "Failed requests" /tmp/perf_results.txt | awk '{print $3}')

if (( $(echo "$AVG_TIME > 500" | bc -l) )); then
    echo "⚠️  Average response time too high: ${AVG_TIME}ms"
fi
```

## 8. Modern DevOps Trends

### Container Orchestration
```
#!/bin/bash
# k8s-rolling-update.sh
DEPLOYMENT=$1
IMAGE=$2
NAMESPACE=${3:-default}

kubectl set image deployment/$DEPLOYMENT $DEPLOYMENT=$IMAGE -n $NAMESPACE
kubectl rollout status deployment/$DEPLOYMENT -n $NAMESPACE

if [ $? -ne 0 ]; then
    kubectl rollout undo deployment/$DEPLOYMENT -n $NAMESPACE
    exit 1
fi
```

### GitOps Workflow
```
#!/bin/bash
# gitops-sync.sh
REPO_URL="https://github.com/company/k8s-manifests.git"
BRANCH="main"
NAMESPACE="production"

git clone $REPO_URL /tmp/manifests
cd /tmp/manifests
git checkout $BRANCH

kubectl apply -f manifests/ -n $NAMESPACE
kubectl get pods -n $NAMESPACE
```

## Common Use Cases Summary:

1. **CI/CD Automation** - Build, test, deploy pipelines
2. **Infrastructure Management** - AWS/Cloud resource management
3. **Monitoring & Alerting** - System health checks, log analysis
4. **Database Operations** - Backups, migrations, maintenance
5. **Security & Compliance** - Vulnerability scans, certificate checks
6. **Development Workflow** - Git hooks, environment setup
7. **Performance Testing** - Load testing, benchmarking
8. **Container Management** - Docker builds, Kubernetes deployments
9. **Configuration Management** - Environment configs, secrets
10. **Disaster Recovery** - Backup automation, failover scripts

These scripts are essential for modern DevOps practices and are used daily in production environments.

### **Key Benefits:**

• **Automation** - Reduces manual errors and saves time
• **Consistency** - Standardized operations across environments
• **Integration** - Works seamlessly with modern DevOps tools
• **Portability** - Runs on any Unix-like system
• **Debugging** - Easy to troubleshoot and modify
