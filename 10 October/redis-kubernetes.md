# Redis Kubernetes

![redis_logo](../assets/redis_logo.png)

Redis on Kubernetes

## Table of Contents

- [What is Redis?](#what-is-redis)
- [Redis Architecture](#redis-architecture)
  - [Standalone Mode](#standalone-mode)
  - [Replication Mode](#replication-mode)
  - [Sentinel Mode](#sentinel-mode)
- [Deploying Redis on Kubernetes](#deploying-redis-on-kubernetes)
  - [Configuration Files](#configuration-files)
  - [Argo CD Application Manifest](#argo-cd-application-manifest)
- [Understanding the Setup](#understanding-the-setup)
  - [Why Use Redis Sentinel?](#why-use-redis-sentinel)
  - [High Availability and Failover](#high-availability-and-failover)
  - [Metrics and Monitoring](#metrics-and-monitoring)
- [Conclusion](#conclusion)

## What is Redis?

Redis is an open-source, in-memory data structure store used as a database, cache, and message broker. Renowned for its speed and versatility, Redis supports various data structures like strings, hashes, lists, sets, and more. It’s widely utilized for caching, real-time analytics, messaging, and session management.

## Redis Architecture

Redis offers multiple architectures to cater to different use cases, from simple caching to complex distributed systems.

### Standalone Mode

- Description: A single Redis instance handling all read and write operations.
- Use Case: Suitable for development or low-load environments where high availability isn’t critical.
- Limitations: Single point of failure; no automatic failover or replication.

### Replication Mode

- Description: A master-slave setup where the master handles write operations and slaves handle read operations.
- Use Case: Improves read scalability and provides basic redundancy.
- Limitations: Failover isn’t automatic; manual intervention is needed if the master fails.

### Sentinel Mode

- Description: Adds Redis Sentinel nodes to monitor master and slave instances, providing automatic failover.
- Use Case: High availability environments where automatic recovery from failures is essential.
- Benefits:
  - Automatic master election and failover.
  - Monitoring and notifications.
  - Configuration provider for clients.

## Deploying Redis on Kubernetes

Deploying Redis on Kubernetes involves containerizing Redis instances and managing them using Kubernetes resources. Helm charts simplify this process by providing pre-configured templates for deploying complex applications.

In this setup, we’re using:

- Argo CD: A declarative, GitOps continuous delivery tool for Kubernetes.
- Bitnami Redis Helm Chart: A widely used chart that supports various Redis architectures, including Sentinel.

## Configuration Files

### Argo CD Application Manifest

File: redis-application.yaml

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: redis
spec:
  project: smartclaims
  source:
    chart: redis
    repoURL: https://charts.bitnami.com/bitnami
    targetRevision: 19.6.1
    helm:
      releaseName: redis
      valuesObject:
        architecture: replication
        metrics:
          enabled: true
          serviceMonitor:
            enabled: true
        auth:
          enabled: false
          sentinel: false
        sentinel:
          enabled: true
  destination:
    server: "https://kubernetes.default.svc"
    namespace: dev
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
      - PruneLast=true
    automated:
      prune: true
      selfHeal: true
```

Explanation:

- apiVersion: Specifies the API version for the Argo CD application.
- kind: Defines the resource type as an Application.
- metadata:
  - name: The name of the application within Argo CD.
- spec:
  - project: Associates the application with a specific Argo CD project.
  - source:
    - chart: Specifies the Helm chart to deploy (redis).
    - repoURL: The repository URL where the chart is located.
    - targetRevision: The version of the chart to use.
    - helm:
      - releaseName: The name given to the Helm release.
      - valuesObject: Custom values to override default chart configurations.
        - architecture: Set to replication for a master-slave setup.
        - metrics: Enables Prometheus metrics and ServiceMonitor for monitoring.
        - auth:
          - enabled: Disables password authentication for simplicity.
          - sentinel: Disables authentication for Sentinel nodes.
        - sentinel:
          - enabled: Enables Redis Sentinel for automatic failover.
  - destination:
    - server: Kubernetes API server address (default in-cluster).
    - namespace: The Kubernetes namespace where Redis will be deployed.
  - syncPolicy:
    - syncOptions:
      - CreateNamespace: Automatically creates the namespace if it doesn’t exist.
      - PruneLast: Deletes resources not defined in the source after syncing.
    - automated:
      - prune: Enables automatic pruning of orphaned resources.
      - selfHeal: Automatically syncs cluster state to the desired state.

## Understanding the Setup

### Why Use Redis Sentinel?

- High Availability: Sentinel provides automatic failover by promoting a slave to master if the master fails.
- Monitoring: Continuously checks the health of master and slave instances.
- Notification: Alerts administrators or systems about changes or failures.
- Configuration Provider: Clients can query Sentinel to find the current master.

### High Availability and Failover

In this setup:

- Master-Slave Replication:
  - The master handles write operations.
  - Slaves replicate data from the master and handle read operations.
- Sentinel Nodes:
  - Deployed alongside Redis instances.
  - Monitor the health of the Redis cluster.
  - Coordinate automatic failover by electing a new master if needed.

Benefits:

- Zero Downtime: Automatic failover minimizes downtime during failures.
- Scalability: Read operations can be scaled horizontally by adding more slaves.
- Data Consistency: Replication ensures slaves have the latest data from the master.

Considerations:

- Data Persistence: Ensure persistent volumes are configured for data durability.
- Network Policies: Configure appropriate network policies for secure communication.
- Authentication: Disabled in this setup but should be enabled in production environments.

### Metrics and Monitoring

- Prometheus Metrics:
  - Enabled via metrics.enabled: true.
  - Exposes Redis metrics in the Prometheus format.
- ServiceMonitor:
  - Enabled via serviceMonitor.enabled: true.
  - Allows Prometheus Operator to discover and scrape metrics automatically.
- Benefits:
  - Visibility: Real-time insights into Redis performance and health.
  - Alerting: Set up alerts based on metrics for proactive issue resolution.

## Conclusion

Deploying Redis with Sentinel on Kubernetes provides a robust, high-availability solution for caching and data storage. By leveraging Argo CD and Helm charts, the deployment process becomes streamlined and manageable. The configuration allows for automatic failover, scalability, and easy integration with monitoring tools.

Citations

- <https://redis.io/>
- <https://redis.io/topics/sentinel>
- <https://charts.bitnami.com/bitnami/redis>
- <https://argo-cd.readthedocs.io/en/stable/>
- <https://kubernetes.io/>
- <https://prometheus.io/docs/>
