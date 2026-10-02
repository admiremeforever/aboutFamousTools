KUBERNETES – INTERVIEW ANSWERS
==============================


BASIC
-----

1. What is Kubernetes?
- Kubernetes is an open-source container orchestration platform.
- It automates deployment, scaling, networking and management of containers.

2. Why do we need Kubernetes?
- Deploy and manage containers.
- Maintain desired number of instances.
- Automatic restart/replacement of failed Pods.
- Service discovery and networking.
- Scaling.
- Rolling deployments and rollback.

3. What is a Kubernetes Cluster?
- A Kubernetes Cluster is a group of machines running Kubernetes.
- It consists of:
  - Control Plane
  - Worker Nodes

4. What are the main components of Kubernetes?
Control Plane:
- API Server
- Scheduler
- Controller Manager
- etcd

Worker Node:
- kubelet
- Container Runtime
- kube-proxy

5. What is Control Plane?
- Control Plane manages the Kubernetes cluster.
- It maintains desired state and makes scheduling/orchestration decisions.

Main components:
- API Server
- Scheduler
- Controller Manager
- etcd

6. What is a Worker Node?
- Worker Node is a machine where application Pods run.
- It provides CPU, memory and other resources.

Main components:
- kubelet
- Container Runtime
- kube-proxy

7. What is a Pod?
- Pod is the smallest deployable unit in Kubernetes.
- It contains one or more containers.
- Containers inside the same Pod share network namespace and can share storage.

8. Can a Pod contain multiple containers?
- Yes.
- Containers in the same Pod:
  - Share the same network namespace.
  - Can communicate using localhost.
  - Can share volumes.
- Usually used for tightly coupled containers such as sidecars.

9. What is a Deployment?
- Deployment manages application Pods.
- It maintains the desired number of replicas.
- Supports:
  - Scaling
  - Rolling updates
  - Rollbacks

10. What is a ReplicaSet?
- ReplicaSet ensures that the required number of Pod replicas are running.
- Deployment normally creates and manages ReplicaSets.

11. What is a Service?
- Service provides a stable network endpoint for a set of Pods.
- Pods can be created/destroyed dynamically, so their IPs can change.
- Service provides stable access to them.

12. What is Namespace?
- Namespace logically separates resources inside a cluster.
- Useful for:
  - Environment separation
  - Team separation
  - Resource management

Example:

dev
prod
monitoring

13. What is Ingress?
- Ingress provides HTTP/HTTPS routing to Kubernetes Services.
- It can route traffic based on:
  - Host
  - URL path

Example:

api.example.com
      ↓
   Ingress
      ↓
   Service
      ↓
     Pods

14. What is ConfigMap?
- ConfigMap stores non-sensitive configuration.
- Example:
  - Environment variables
  - Application properties
  - Configuration files

15. What is Secret?
- Secret stores sensitive configuration such as:
  - Passwords
  - Tokens
  - Credentials
- It can be exposed to Pods as environment variables or mounted files.


POD / DEPLOYMENT
----------------

16. What happens when a Pod crashes?
- Kubernetes detects that the desired state is not maintained.
- If managed by Deployment/ReplicaSet, a replacement Pod is created.
- If the container crashes inside the Pod, kubelet may restart the container according to its restart policy.

17. What happens when a Worker Node goes down?
- Pods running on that node become unavailable.
- Kubernetes detects node failure.
- Controllers try to create replacement Pods.
- Scheduler places replacement Pods on healthy nodes if resources are available.

18. How does Kubernetes maintain the desired number of Pods?
Example:

Desired replicas = 3
Running Pods = 2

Controller detects:

Desired state != Current state

Then creates another Pod.

This is the basic Kubernetes reconciliation model.

19. Deployment vs ReplicaSet?

Deployment:
- Higher-level abstraction.
- Manages ReplicaSets.
- Supports rolling updates and rollback.

ReplicaSet:
- Maintains desired number of Pods.
- Does not provide the deployment/rollback functionality itself.

20. Deployment vs Pod?

Pod:
- Runs containers.
- Represents the actual running workload.

Deployment:
- Manages Pods.
- Maintains replicas.
- Handles updates and rollback.

21. How do you scale a Deployment?

Command:

kubectl scale deployment <name> --replicas=5

Or configure:

replicas: 5

22. What is Rolling Deployment?
- Gradually replaces old Pods with new Pods.
- Old version and new version can temporarily coexist.
- Helps achieve zero/minimal downtime.

Example:

Old:
P1 P2 P3

New:
P1 P2 P3 P4

Then old Pods are gradually removed.

23. How do you perform Rollback?

Check history:

kubectl rollout history deployment <name>

Rollback:

kubectl rollout undo deployment <name>

24. What is a Readiness Probe?
- Determines whether a Pod is ready to receive traffic.
- If readiness fails, the Pod is removed from Service endpoints.
- Container can continue running.

25. What is a Liveness Probe?
- Determines whether a container is still healthy.
- If liveness fails repeatedly, Kubernetes restarts the container.

26. Readiness vs Liveness Probe?

Readiness:
- "Can this Pod receive traffic?"
- Failure → remove from Service endpoints.

Liveness:
- "Is this container still healthy?"
- Failure → restart container.

27. What is CrashLoopBackOff?
- Container starts and repeatedly crashes.
- Kubernetes repeatedly tries to restart it.
- Backoff is introduced between restart attempts.

Common causes:
- Application startup failure.
- Wrong configuration.
- Missing environment variable.
- Dependency failure.
- Incorrect command.
- OOM.

28. What is ImagePullBackOff?
- Kubernetes cannot pull the container image.
- Kubernetes retries with increasing delay.

Common causes:
- Wrong image name/tag.
- Image doesn't exist.
- Private registry authentication failure.
- Registry/network issue.

29. Why does a Pod remain in Pending state?
Common reasons:
- Insufficient CPU/memory.
- No suitable node.
- Node constraints.
- Taints/tolerations mismatch.
- Affinity/anti-affinity rules.
- PVC/storage issue.

Command:

kubectl describe pod <pod-name>

Check Events.

30. What is OOMKilled?
- Container exceeded its memory limit.
- Kubernetes/container runtime kills the container.

Check:
- Memory limit.
- JVM heap.
- Memory leak.
- Large objects/requests.
- Cache.
- Thread count.


SERVICES / NETWORKING
--------------------

31. Why do we need a Kubernetes Service?
- Pod IPs are dynamic.
- Pods can be recreated.
- Service provides:
  - Stable IP/DNS.
  - Load balancing across selected Pods.
  - Stable access to application.

32. What are different types of Services?

Main types:
- ClusterIP
- NodePort
- LoadBalancer
- ExternalName

33. ClusterIP vs NodePort vs LoadBalancer?

ClusterIP:
- Default.
- Internal cluster access.

NodePort:
- Exposes Service on a port of each Node.
- Can be accessed using NodeIP:NodePort.

LoadBalancer:
- Exposes Service externally.
- In cloud environments, typically provisions/integrates with a cloud Load Balancer.

34. What is Service Discovery?
- Kubernetes allows Services to be discovered using DNS.
- Applications can communicate using Service name instead of Pod IP.

Example:

http://sync-service:8080

35. How does one Pod communicate with another Pod?
- Kubernetes networking allows Pods to communicate using Pod IPs.
- Usually applications communicate through a Service rather than directly using Pod IPs.

Example:

Pod A
 ↓
Service B
 ↓
Pod B

36. How does a Service find its Pods?
- Service uses label selectors.

Example:

Service selector:
app: sync-service

Pods:

app: sync-service

Those matching Pods become Service endpoints.

37. What is a Service Selector?
- Selector defines which Pods belong to a Service.

Example:

selector:
  app: sync-service

Service routes traffic to Pods having:

app=sync-service

38. What is Ingress?
- Ingress defines HTTP/HTTPS routing rules.
- It routes external traffic to Kubernetes Services.
- Requires an Ingress Controller to implement the routing.

39. Ingress vs Service?

Service:
- Provides networking/load balancing to Pods.

Ingress:
- Provides HTTP/HTTPS routing to Services.

Example:

Internet
   ↓
Ingress
   ↓
Service
   ↓
Pods

40. How does Internet traffic reach a Pod?

Typical flow:

Internet
   ↓
Load Balancer
   ↓
Ingress
   ↓
Service
   ↓
Pod


CONFIGURATION / STORAGE
-----------------------

41. ConfigMap vs Secret?

ConfigMap:
- Non-sensitive configuration.

Secret:
- Sensitive data such as credentials/tokens.

Both can be:
- Environment variables.
- Mounted as files.

42. How do you pass environment variables to Pods?

Using:

1. Directly in Pod/Deployment YAML.
2. ConfigMap.
3. Secret.

Example:

env:
  - name: DB_HOST
    valueFrom:
      configMapKeyRef:
        name: app-config
        key: DB_HOST

43. What is PersistentVolume (PV)?
- PV represents storage available to the Kubernetes cluster.
- It abstracts the underlying storage.

44. What is PersistentVolumeClaim (PVC)?
- PVC is a request for storage by an application.
- Pod uses PVC to access persistent storage.

Flow:

Pod
 ↓
PVC
 ↓
PV
 ↓
Storage

45. Why do we need persistent storage?
- Pod storage is generally ephemeral.
- Pod deletion/recreation can lose local container data.
- Persistent storage keeps data beyond Pod lifecycle.

Used for:
- Databases.
- Files.
- Persistent application data.

46. What happens to Pod data when the Pod is deleted?
- Data stored only in the container's ephemeral filesystem can be lost.
- Data stored through a PersistentVolume can survive Pod deletion, depending on storage and reclaim configuration.


SCALING
-------

47. How do you scale Pods?
Manual:

kubectl scale deployment app --replicas=5

Automatic:
- HPA
- VPA (different use case)

48. What is HPA?
- Horizontal Pod Autoscaler.
- Automatically changes the number of Pod replicas based on metrics.

Example:

High CPU
  ↓
HPA
  ↓
More Pods

Low CPU
  ↓
HPA
  ↓
Fewer Pods

49. What is VPA?
- Vertical Pod Autoscaler.
- Adjusts CPU/memory resource requests/limits for Pods based on usage/recommendations.

50. HPA vs VPA?

HPA:
- Changes number of Pods.
- Horizontal scaling.

VPA:
- Changes resources allocated to Pods.
- Vertical scaling.

51. How does Kubernetes scale based on CPU?
- Configure CPU requests.
- HPA monitors CPU utilization.
- If utilization exceeds target, HPA increases replicas.
- If utilization decreases, HPA can reduce replicas.

52. What is Cluster Autoscaler?
- Automatically adjusts the number of worker nodes.
- Adds nodes when Pods cannot be scheduled because of insufficient capacity.
- Removes underutilized nodes when possible.

53. HPA vs Cluster Autoscaler?

HPA:
- Scales Pods.

Cluster Autoscaler:
- Scales Worker Nodes.

Example:

Traffic increases
     ↓
HPA creates more Pods
     ↓
No node capacity
     ↓
Cluster Autoscaler adds nodes


RESOURCE MANAGEMENT
-------------------

54. What are CPU/Memory Requests?
- Minimum resource amount requested by a Pod/container.
- Scheduler uses requests to decide where the Pod can run.

55. What are CPU/Memory Limits?
- Maximum resource amount a container is allowed to consume.

56. Request vs Limit?

Request:
- Used for scheduling.
- Represents expected/reserved resource requirement.

Limit:
- Maximum resource usage.

Example:

requests:
  memory: 512Mi

limits:
  memory: 1Gi

57. What happens when a container exceeds its memory limit?
- Container can be terminated with OOMKilled.
- Kubernetes may restart it depending on restart policy.

58. What happens when a container exceeds its CPU limit?
- CPU is throttled.
- Container is generally not killed just because it exceeds CPU limit.

59. Why is a Pod Pending because of insufficient resources?
Example:

Node available:
CPU = 1 core

Pod request:
CPU = 2 cores

Scheduler cannot place the Pod because no node satisfies the resource request.

Result:

Pod → Pending


TROUBLESHOOTING
---------------

60. Pod is not starting. How troubleshoot?

Step 1:
kubectl get pods

Step 2:
kubectl describe pod <pod>

Step 3:
Check Events.

Step 4:
kubectl logs <pod>

Check:
- Image.
- Configuration.
- Secrets.
- Resources.
- Scheduling.
- Probes.
- Storage.
- Dependencies.


61. Pod is Running but application is unreachable.

Check:

Pod
 ↓
Container
 ↓
Service
 ↓
Endpoints
 ↓
Ingress
 ↓
Load Balancer

Check:
- Application listening port.
- Container port.
- Service port.
- targetPort.
- Service selector.
- Endpoints.
- Ingress.
- Load Balancer.
- Security/network rules.

Useful commands:

kubectl get pods
kubectl get svc
kubectl get endpoints
kubectl describe ingress


62. Pod is CrashLoopBackOff. How troubleshoot?

1. Check logs:

kubectl logs <pod>

2. Check previous container logs:

kubectl logs <pod> --previous

3. Describe Pod:

kubectl describe pod <pod>

4. Check:
- Application startup.
- Environment variables.
- ConfigMap/Secret.
- Database/dependency connection.
- Probes.
- Memory limits.
- Command/arguments.


63. Pod is ImagePullBackOff. Why?

Check:
- Image name.
- Image tag.
- Registry availability.
- Registry credentials.
- imagePullSecrets.
- Network connectivity.

Command:

kubectl describe pod <pod>


64. Pod is Pending. Why?

Check:

kubectl describe pod <pod>

Look at Events.

Common causes:
- Insufficient CPU/memory.
- Node unavailable.
- Taints.
- Tolerations.
- Affinity.
- PVC.
- Scheduling constraints.


65. Service is not reaching Pods. What check?

Check:

1. Service selector.
2. Pod labels.
3. Service port.
4. targetPort.
5. Endpoints.
6. Pod readiness.

Example:

Service:
selector:
  app: sync-service

Pod:
labels:
  app: sync-service

If labels don't match → Service won't select the Pod.


66. Deployment rollout is stuck. What check?

Check:

kubectl rollout status deployment <name>

Then:

kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>

Check:
- New Pods starting?
- Readiness probe failing?
- Image pull?
- Application startup?
- Resource limits?
- Configuration?
- Dependency connectivity?


KUBECTL
-------

67. How do you check Pods?

kubectl get pods

68. How do you check Pods across namespaces?

kubectl get pods -A

69. How do you get detailed Pod information?

kubectl describe pod <pod-name>

70. How do you check application logs?

kubectl logs <pod-name>

For previous crashed container:

kubectl logs <pod-name> --previous

71. How do you execute a command inside a Pod?

kubectl exec -it <pod-name> -- /bin/sh

72. How do you check Services?

kubectl get svc

73. How do you check Deployments?

kubectl get deployments

74. How do you scale a Deployment?

kubectl scale deployment <name> --replicas=3

75. How do you restart a Deployment?

kubectl rollout restart deployment <name>

76. How do you rollback a Deployment?

kubectl rollout undo deployment <name>

77. How do you check rollout status?

kubectl rollout status deployment <name>


KUBERNETES ARCHITECTURE
-----------------------

78. What are the Control Plane components?

1. API Server
2. Scheduler
3. Controller Manager
4. etcd

79. What does API Server do?
- Entry point for Kubernetes API.
- Receives requests from kubectl, controllers and other clients.
- Validates and processes API requests.
- Stores/reads cluster state through etcd.

80. What does Scheduler do?
- Watches for newly created unscheduled Pods.
- Selects a suitable Worker Node.
- Considers:
  - CPU/memory requests.
  - Taints/tolerations.
  - Affinity.
  - Other scheduling constraints.

81. What does Controller Manager do?
- Runs Kubernetes controllers.
- Continuously compares desired state with current state.
- Takes actions to reconcile the difference.

Example:

Desired replicas = 3
Current replicas = 2

Controller creates another Pod.

82. What is etcd?
- Distributed key-value store.
- Stores Kubernetes cluster state/configuration.
- It is a critical component of the Control Plane.


SCENARIO QUESTIONS
------------------

83. A Pod crashes. What happens?

- Kubernetes detects container/Pod failure.
- If container crashes, kubelet may restart it.
- If Pod is managed by Deployment/ReplicaSet and Pod is lost, replacement Pod is created.
- Service routes traffic to healthy/ready Pods.

84. A Worker Node crashes. What happens?

Node failure
    ↓
Pods become unavailable
    ↓
Kubernetes detects failure
    ↓
Replacement Pods
    ↓
Scheduler places them on healthy nodes

Requires:
- Available capacity.
- Appropriate Deployment/ReplicaSet.
- Scheduling constraints allowing placement.


85. New version deployed and Pods keep crashing. What do you do?

1. Check:

kubectl get pods

2. Check logs:

kubectl logs <pod>

3. Check previous logs:

kubectl logs <pod> --previous

4. Describe:

kubectl describe pod <pod>

5. Check:
- Configuration.
- Environment variables.
- Secrets.
- Image.
- Dependencies.
- Resource limits.
- Probes.

6. If production impact → rollback:

kubectl rollout undo deployment <name>


86. Application works inside Pod but not from outside. How troubleshoot?

Check:

Application
    ↓
Pod Port
    ↓
Service
    ↓
Ingress
    ↓
Load Balancer

Check:
- Application listening address/port.
- Container port.
- Service targetPort.
- Service selector.
- Endpoints.
- Ingress rules.
- Load Balancer.
- Network/Security rules.


87. Application receives high traffic. How scale?

1. Configure multiple replicas.
2. Configure HPA.
3. Monitor CPU/memory/custom metrics.
4. Scale Worker Nodes if required.

Traffic
  ↓
HPA
  ↓
More Pods
  ↓
More Nodes if required


88. How achieve zero-downtime deployment?

- Multiple replicas.
- RollingUpdate.
- Readiness probe.
- Proper maxUnavailable.
- Proper maxSurge.
- Graceful application shutdown.

Traffic should only go to Ready Pods.


89. How deploy 3 replicas across different nodes?

Use Pod anti-affinity or topology spread constraints.

Goal:

Node-1 → Pod-1
Node-2 → Pod-2
Node-3 → Pod-3

This improves failure tolerance.

90. How expose application to Internet?

Typical:

Internet
   ↓
Load Balancer
   ↓
Ingress
   ↓
Service
   ↓
Pods

Use appropriate cloud Load Balancer/Ingress Controller configuration.


91. How make application highly available?

- Multiple Pod replicas.
- Spread Pods across nodes/AZs.
- Multiple worker nodes.
- HPA.
- Node autoscaling.
- Readiness/Liveness probes.
- Load Balancing.
- Rolling deployments.
- HA dependencies/database where required.


92. How restrict application to only inside cluster?

Use:

ClusterIP Service

Example:

Internal clients
      ↓
ClusterIP Service
      ↓
Pods

Do not expose it through public LoadBalancer/NodePort.


93. Application uses too much memory. How troubleshoot?

Check:

1. Pod memory usage.
2. Container memory limit.
3. JVM heap.
4. GC.
5. Memory leak.
6. Cache.
7. Large objects.
8. Thread count.
9. Request/response size.

If OOMKilled:

kubectl describe pod <pod>


94. Kubernetes node has insufficient resources. What happens?

If a new Pod cannot fit:

Pod → Pending

Check:

kubectl describe pod <pod>

If Cluster Autoscaler/Karpenter is configured:
- Additional Worker Node may be provisioned.
- Pod can then be scheduled.


95. How rollback bad production deployment?

Check:

kubectl rollout status deployment <name>

Rollback:

kubectl rollout undo deployment <name>

Verify:

kubectl rollout status deployment <name>

Then investigate the failed release.


96. How troubleshoot intermittent Pod failures?

Check:
- Pod restarts.
- Application logs.
- Previous container logs.
- OOMKilled.
- CPU throttling.
- Memory usage.
- Liveness/readiness probes.
- Node health.
- Network issues.
- DB/Redis/Kafka dependencies.
- Recent deployments.

Commands:

kubectl get pods
kubectl describe pod <pod>
kubectl logs <pod>
kubectl logs <pod> --previous
kubectl get events



====================================
shortcut Revision 

1. Pod vs Deployment vs ReplicaSet
2. ClusterIP vs NodePort vs LoadBalancer
3. Ingress
4. Service Discovery
5. Readiness vs Liveness
6. Rolling Deployment + Rollback
7. CrashLoopBackOff
8. ImagePullBackOff
9. Pending Pod
10. OOMKilled
11. Requests vs Limits
12. HPA
13. HPA vs Cluster Autoscaler
14. Pod networking
15. ConfigMap vs Secret
16. PV vs PVC
17. Control Plane components
18. Worker Node failure
19. Zero-downtime deployment
20. Kubernetes troubleshooting
21. HA architecture
