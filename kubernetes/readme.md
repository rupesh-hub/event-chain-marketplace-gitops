# Kubernetes Commands - Production Guide

## Table of Contents
1. [Prerequisites & Setup](#prerequisites--setup)
2. [Namespace & Secrets Management](#namespace--secrets-management)
3. [Config Server](#config-server)
4. [Discovery Server](#discovery-server)
5. [Product Service](#product-service)
6. [Order Service](#order-service)
7. [API Gateway Server](#api-gateway-server)
8. [Frontend](#frontend)
9. [Monitoring & Debugging](#monitoring--debugging)
10. [Production Best Practices](#production-best-practices)
11. [Cleanup & Scaling](#cleanup--scaling)

---

## Prerequisites & Setup

### Create Namespace
```bash
# Create dedicated namespace for the application
kubectl create namespace ecm
kubectl label namespace ecm environment=production
```

### Verify Cluster Connection
```bash
# Check cluster info and connection
kubectl cluster-info
kubectl get nodes
kubectl get nodes -o wide  # Detailed node information
```

---

## Namespace & Secrets Management

### GitHub Credentials (Private Repository)
```bash
# Create secret for private GitHub repository access
kubectl create secret generic git-credentials \
  --from-literal=username=<your-github-username> \
  --from-literal=password=<your-github-personal-access-token> \
  --namespace=ecm

# Example with actual values


# Verify secret creation
kubectl get secret git-credentials -n ecm
kubectl describe secret git-credentials -n ecm
```

### Configuration Management
```bash
# View configuration maps
kubectl get configmap ecm-configs -n ecm
kubectl describe configmap ecm-configs -n ecm

# Create ConfigMap from file
kubectl create configmap ecm-configs --from-file=<config-file> -n ecm

# Update ConfigMap
kubectl apply -f configmap.yaml -n ecm
```

---

## Config Server

### Deployment Management
```bash
# Deploy config server
kubectl apply -f config-server-deployment.yaml -n ecm

# Check deployment status
kubectl get deployment config-server-deployment -n ecm
kubectl describe deployment config-server-deployment -n ecm
kubectl rollout status deployment/config-server-deployment -n ecm

# View logs
kubectl logs -f deployment/config-server-deployment -n ecm
kubectl logs deployment/config-server-deployment -n ecm --tail=50
```

### Service Management
```bash
# Apply config server service
kubectl apply -f config-server-service.yaml -n ecm

# Check service
kubectl get svc config-server -n ecm
kubectl describe svc config-server -n ecm

# Port forward for local access
kubectl port-forward svc/config-server -n ecm 8071:8071 --address=0.0.0.0 &
```

### Health Checks
```bash
# Check config server health
kubectl exec -it deployment/config-server-deployment -n ecm -- curl http://localhost:8071/health

# Rolling update (if new image)
kubectl set image deployment/config-server-deployment \
  config-server=your-registry/config-server:v2 -n ecm --record

# Rollback if needed
kubectl rollout undo deployment/config-server-deployment -n ecm
```

### Cleanup
```bash
# Delete config server deployment
kubectl delete deployment config-server-deployment -n ecm

# Delete service
kubectl delete svc config-server -n ecm

# Delete all config server related resources
kubectl delete -f config-server-deployment.yaml -n ecm
kubectl delete -f config-server-service.yaml -n ecm
```

---

## Discovery Server

### Deployment Management
```bash
# Deploy discovery server (Eureka)
kubectl apply -f discovery-server-deployment.yaml -n ecm

# Check deployment status
kubectl get deployment discovery-server-deployment -n ecm
kubectl describe deployment discovery-server-deployment -n ecm
kubectl rollout status deployment/discovery-server-deployment -n ecm

# View logs
kubectl logs -f deployment/discovery-server-deployment -n ecm
kubectl logs deployment/discovery-server-deployment -n ecm --all-containers=true
```

### Service Management
```bash
# Apply discovery server service
kubectl apply -f discovery-server-service.yaml -n ecm

# Check service details
kubectl get svc discovery-server -n ecm -o wide
kubectl describe svc discovery-server -n ecm

# Port forward for UI access
kubectl port-forward svc/discovery-server -n ecm 8061:8061 --address=0.0.0.0 &
```

### Health & Monitoring
```bash
# Check registered instances
kubectl exec -it deployment/discovery-server-deployment -n ecm -- curl http://localhost:8061/eureka/apps

# Check specific service registration
kubectl exec -it deployment/discovery-server-deployment -n ecm -- curl http://localhost:8061/eureka/apps/<SERVICE-NAME>
```

### Cleanup
```bash
# Delete discovery server resources
kubectl delete -f discovery-server-deployment.yaml -n ecm
kubectl delete -f discovery-server-service.yaml -n ecm
```

---

## Product Service

### Deployment Management
```bash
# Deploy product service
kubectl apply -f product-service-deployment.yaml -n ecm

# Check deployment
kubectl get deployment product-svc-deployment -n ecm
kubectl describe deployment product-svc-deployment -n ecm
kubectl get pods -l app=product-service -n ecm

# View logs
kubectl logs -f deployment/product-svc-deployment -n ecm
kubectl logs -f deployment/product-svc-deployment -n ecm --previous  # Previous crashed pod logs
```

### Service Management
```bash
# Apply product service service
kubectl apply -f product-service-svc.yaml -n ecm

# Check service endpoint
kubectl get svc product-service -n ecm
kubectl get endpoints product-service -n ecm

# Port forward
kubectl port-forward svc/product-service -n ecm 8181:8181 --address=0.0.0.0 &
```

### Scaling & Performance
```bash
# Scale product service replicas
kubectl scale deployment product-svc-deployment --replicas=3 -n ecm

# Check HPA if configured
kubectl get hpa -n ecm
kubectl describe hpa product-svc-deployment -n ecm

# Monitor resource usage
kubectl top deployment product-svc-deployment -n ecm
kubectl top pods -l app=product-service -n ecm
```

### Database Connectivity
```bash
# Check database connectivity
kubectl exec -it deployment/product-svc-deployment -n ecm -- curl http://localhost:8181/health

# Verify environment variables
kubectl exec -it deployment/product-svc-deployment -n ecm -- env | grep DB
```

### Cleanup
```bash
# Delete product service
kubectl delete -f product-service-deployment.yaml -n ecm
kubectl delete -f product-service-svc.yaml -n ecm
```

---

## Order Service

### Deployment Management
```bash
# Deploy order service
kubectl apply -f order-service-deployment.yaml -n ecm

# Check deployment status
kubectl get deployment order-svc-deployment -n ecm
kubectl describe deployment order-svc-deployment -n ecm
kubectl rollout status deployment/order-svc-deployment -n ecm

# View logs (follow mode for real-time)
kubectl logs -f deployment/order-svc-deployment -n ecm
kubectl logs deployment/order-svc-deployment -n ecm --tail=100 --timestamps=true
```

### Service Management
```bash
# Apply order service service
kubectl apply -f order-service-svc.yaml -n ecm

# Check service availability
kubectl get svc order-service -n ecm
kubectl get ep order-service -n ecm
```

### Port Forwarding
```bash
# Port forward for local testing
kubectl port-forward svc/order-service -n ecm 8182:8182 --address=0.0.0.0 &
```

### Debugging & Troubleshooting
```bash
# Check pod status
kubectl get pods -l app=order-service -n ecm -o wide

# Describe pod for issues
kubectl describe pod <pod-name> -n ecm

# SSH into pod for debugging
kubectl exec -it <pod-name> -n ecm -- /bin/bash

# Check logs for errors
kubectl logs deployment/order-svc-deployment -n ecm --since=5m
```

### Cleanup
```bash
# Delete order service deployment
kubectl delete deployment order-svc-deployment -n ecm

# Delete order service
kubectl delete svc order-service -n ecm

# Delete all resources
kubectl delete -f order-service-deployment.yaml -n ecm
kubectl delete -f order-service-svc.yaml -n ecm
```

---

## API Gateway Server

### Deployment Management
```bash
# Deploy API gateway
kubectl apply -f gateway-server-deployment.yaml -n ecm

# Check deployment
kubectl get deployment gateway-server-deployment -n ecm
kubectl describe deployment gateway-server-deployment -n ecm
kubectl get pods -l app=gateway-server -n ecm -o wide

# Monitor rollout
kubectl rollout status deployment/gateway-server-deployment -n ecm
```

### Service Management
```bash
# Apply gateway service
kubectl apply -f gateway-server-service.yaml -n ecm

# Verify service
kubectl get svc gateway-server -n ecm
kubectl get svc gateway-server -n ecm -o yaml

# Check DNS resolution
kubectl exec -it deployment/gateway-server-deployment -n ecm -- nslookup gateway-server.ecm.svc.cluster.local
```

### Access & Routing
```bash
# Port forward gateway
kubectl port-forward svc/gateway-server -n ecm 8072:8072 --address=0.0.0.0 &

# Test gateway endpoints
kubectl exec -it deployment/gateway-server-deployment -n ecm -- curl http://localhost:8072/health

# Check routing configuration
kubectl get configmap gateway-config -n ecm
kubectl describe configmap gateway-config -n ecm
```

### Load Balancing
```bash
# Check endpoint distribution
kubectl get endpoints gateway-server -n ecm -o yaml

# Scale gateway replicas
kubectl scale deployment gateway-server-deployment --replicas=3 -n ecm
```

### Cleanup
```bash
# Delete gateway resources
kubectl delete -f gateway-server-deployment.yaml -n ecm
kubectl delete -f gateway-server-service.yaml -n ecm
```

---

## Frontend

### Deployment Management
```bash
# Deploy frontend application
kubectl apply -f frontend-deployment.yaml -n ecm

# Check deployment status
kubectl get deployment frontend-deployment -n ecm
kubectl describe deployment frontend-deployment -n ecm
kubectl rollout status deployment/frontend-deployment -n ecm

# View frontend logs
kubectl logs -f deployment/frontend-deployment -n ecm
kubectl logs deployment/frontend-deployment -n ecm --container=frontend
```

### Service Management
```bash
# Apply frontend service
kubectl apply -f frontend-service.yaml -n ecm

# Check service details
kubectl get svc frontend-service -n ecm
kubectl describe svc frontend-service -n ecm

# Get LoadBalancer external IP (if using cloud provider)
kubectl get svc frontend-service -n ecm -o wide
```

### Accessing the Application
```bash
# Port forward for local access
kubectl port-forward svc/frontend-service -n ecm 8080:8080 --address=0.0.0.0 &

# Test frontend connectivity
kubectl exec -it deployment/frontend-deployment -n ecm -- curl http://localhost:8080

# Get Ingress details (if using Ingress)
kubectl get ingress -n ecm
kubectl describe ingress frontend-ingress -n ecm
```

### Static Assets & Caching
```bash
# Check build artifacts in pod
kubectl exec -it deployment/frontend-deployment -n ecm -- ls -la /usr/share/nginx/html

# Verify image served correctly
kubectl exec -it deployment/frontend-deployment -n ecm -- cat /etc/nginx/nginx.conf
```

### Scaling & Updates
```bash
# Scale frontend replicas
kubectl scale deployment frontend-deployment --replicas=3 -n ecm

# Rolling update with new image
kubectl set image deployment/frontend-deployment \
  frontend=your-registry/frontend:v2 -n ecm --record

# Check rolling update status
kubectl rollout status deployment/frontend-deployment -n ecm

# View rollout history
kubectl rollout history deployment/frontend-deployment -n ecm
```

### Cleanup
```bash
# Delete frontend resources
kubectl delete -f frontend-deployment.yaml -n ecm
kubectl delete -f frontend-service.yaml -n ecm
```

---

## Monitoring & Debugging

### Pod & Container Management
```bash
# List all pods in namespace
kubectl get pods -n ecm
kubectl get pods -n ecm -o wide
kubectl get pods -n ecm --show-labels

# Describe specific pod
kubectl describe pod <pod-name> -n ecm

# Get pod logs
kubectl logs <pod-name> -n ecm
kubectl logs <pod-name> -n ecm --previous  # From crashed container
kubectl logs <pod-name> -n ecm -c <container-name>  # Specific container

# Real-time log streaming
kubectl logs -f <pod-name> -n ecm
kubectl logs -f <pod-name> -n ecm --all-containers=true
```

### Resource Monitoring
```bash
# Check CPU and Memory usage
kubectl top nodes
kubectl top pods -n ecm
kubectl top deployment <deployment-name> -n ecm

# Get resource requests and limits
kubectl describe nodes
kubectl describe pod <pod-name> -n ecm | grep -A 5 "Limits\|Requests"
```

### Events & Status
```bash
# View cluster events
kubectl get events -n ecm --sort-by='.lastTimestamp'
kubectl describe event <event-name> -n ecm

# Check deployment events
kubectl describe deployment <deployment-name> -n ecm

# Get namespace status
kubectl describe namespace ecm
```

### Network Debugging
```bash
# Test service DNS resolution
kubectl exec -it <pod-name> -n ecm -- nslookup <service-name>

# Test connectivity between services
kubectl exec -it <pod-name> -n ecm -- curl http://<service-name>:8080

# Check network policies
kubectl get networkpolicies -n ecm
```

### Shell Access & Inspection
```bash
# Execute command in pod
kubectl exec -it <pod-name> -n ecm -- /bin/bash
kubectl exec -it <pod-name> -n ecm -- sh

# Copy files from pod
kubectl cp <pod-name>:/path/to/file ./local-file -n ecm

# Copy files to pod
kubectl cp ./local-file <pod-name>:/path/to/file -n ecm
```

### Persistent Volume Management
```bash
# List PVs and PVCs
kubectl get pv
kubectl get pvc -n ecm
kubectl describe pvc <pvc-name> -n ecm

# Check PV usage
kubectl exec -it <pod-name> -n ecm -- df -h
```

---

## Production Best Practices

### Health Checks & Readiness Probes
```bash
# Verify liveness and readiness probes are configured
kubectl get deployment <deployment-name> -n ecm -o yaml | grep -A 10 "livenessProbe\|readinessProbe"

# Test health endpoints manually
kubectl exec -it deployment/<deployment-name> -n ecm -- curl http://localhost:<port>/health

# Check probe failures
kubectl describe pod <pod-name> -n ecm | grep -A 5 "Liveness\|Readiness"
```

### Resource Management
```bash
# Set resource requests and limits (edit deployment)
# In deployment yaml:
# resources:
#   requests:
#     cpu: "500m"
#     memory: "512Mi"
#   limits:
#     cpu: "1000m"
#     memory: "1Gi"

# Apply resource policies
kubectl apply -f resource-quota.yaml -n ecm
kubectl get resourcequota -n ecm
kubectl describe resourcequota <quota-name> -n ecm
```

### Autoscaling
```bash
# Create Horizontal Pod Autoscaler
kubectl autoscale deployment <deployment-name> --min=2 --max=10 -n ecm

# Check HPA status
kubectl get hpa -n ecm
kubectl describe hpa <hpa-name> -n ecm
kubectl watch hpa <hpa-name> -n ecm

# Apply custom HPA from yaml
kubectl apply -f hpa.yaml -n ecm
```

### Security Best Practices
```bash
# Check security context
kubectl get deployment <deployment-name> -n ecm -o yaml | grep -A 5 "securityContext"

# Verify RBAC roles
kubectl get roles -n ecm
kubectl get rolebindings -n ecm
kubectl describe role <role-name> -n ecm

# Check network policies
kubectl get networkpolicies -n ecm
kubectl describe networkpolicy <policy-name> -n ecm
```

### Backup & Disaster Recovery
```bash
# Backup deployment configuration
kubectl get deployment <deployment-name> -n ecm -o yaml > deployment-backup.yaml

# Backup all namespace resources
kubectl get all -n ecm -o yaml > ecm-backup.yaml

# Verify backup
cat deployment-backup.yaml
```

---

## Cleanup & Scaling

### Temporary Cleanup (Development)
```bash
# Delete specific deployment
kubectl delete deployment <deployment-name> -n ecm

# Delete specific service
kubectl delete svc <service-name> -n ecm

# Delete pod (will be recreated)
kubectl delete pod <pod-name> -n ecm
```

### Complete Cleanup
```bash
# Delete entire namespace (removes all resources)
kubectl delete namespace ecm

# Verify namespace deletion
kubectl get namespace

# Force delete stuck namespace
kubectl delete namespace ecm --ignore-not-found=true --grace-period=0 --force
```

### Scaling Operations
```bash
# Scale deployment up
kubectl scale deployment <deployment-name> --replicas=5 -n ecm

# Scale deployment down
kubectl scale deployment <deployment-name> --replicas=1 -n ecm

# Patch deployment replicas
kubectl patch deployment <deployment-name> -p '{"spec":{"replicas":3}}' -n ecm
```

### Rolling Updates & Rollbacks
```bash
# Update deployment image
kubectl set image deployment/<deployment-name> \
  <container-name>=new-image:tag -n ecm --record

# Check rollout history
kubectl rollout history deployment/<deployment-name> -n ecm

# Rollback to previous version
kubectl rollout undo deployment/<deployment-name> -n ecm

# Rollback to specific revision
kubectl rollout undo deployment/<deployment-name> --to-revision=2 -n ecm

# Pause rollout (during issues)
kubectl rollout pause deployment/<deployment-name> -n ecm

# Resume rollout
kubectl rollout resume deployment/<deployment-name> -n ecm
```

---

## Quick Reference

### Most Used Commands in Production
```bash
# Daily monitoring
kubectl get pods -n ecm -o wide
kubectl logs -f deployment/<deployment-name> -n ecm
kubectl top pods -n ecm
kubectl describe pod <pod-name> -n ecm
kubectl get events -n ecm --sort-by='.lastTimestamp'

# Troubleshooting
kubectl describe deployment <deployment-name> -n ecm
kubectl logs deployment/<deployment-name> -n ecm --tail=50
kubectl exec -it <pod-name> -n ecm -- /bin/bash

# Scaling & Updates
kubectl scale deployment <deployment-name> --replicas=3 -n ecm
kubectl set image deployment/<deployment-name> <container>=image:tag -n ecm
kubectl rollout status deployment/<deployment-name> -n ecm
```

---

**Last Updated:** 2026-02-20
