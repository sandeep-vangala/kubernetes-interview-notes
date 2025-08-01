# kubernetes-interview-notes


# Kubernetes Interview Preparation Notes

Below are detailed notes covering the key topics from a 4-day Kubernetes course, explaining how each concept is achieved in practice, followed by in-depth interview questions and answers tailored for a Cloud DevOps Engineer.

## Day 1: Linux Containers, Kubernetes Fundamentals, and Deployment Strategies

### 1. Linux Containers and Docker
**Overview**: Kubernetes uses Linux containers to package and run applications. Containers provide process isolation, lightweight execution compared to VMs, and consistent environments. Docker is a popular container runtime, but alternatives like containerd exist.

**How Achieved**:
- **Containers vs VMs**: Containers share the host OS kernel, using namespaces (PID, network, mount) and cgroups for isolation, making them lighter than VMs, which require a full OS. Achieved using Docker or containerd to create isolated environments.
- **Process Isolation**: Namespaces ensure processes are isolated (e.g., separate PID trees), while cgroups limit resource usage (CPU, memory). Configured via Docker CLI or container runtime settings.
- **Docker Architecture**: Docker uses a client-server model where the Docker CLI communicates with the Docker daemon (dockerd) via REST API to manage containers, images, and volumes.
- **Running Containers**: Use `docker run` to start containers, specifying image, ports, and volumes. Example: `docker run -d -p 80:80 nginx`.
- **Building Images**: Create Dockerfiles to define application setup (e.g., `FROM node:16`, `COPY`, `RUN npm install`). Build with `docker build -t myapp .`.
- **Mounting Volumes**: Use `-v` or `--mount` to persist data outside containers (e.g., `docker run -v /host:/container nginx`).
- **Exposing Ports**: Map container ports to host ports (e.g., `-p 8080:80`).
- **Container Lifecycle**: Manage with `docker start`, `stop`, `rm`. Use `docker ps` to monitor.
- **Environment Variables**: Inject via `-e` flag (e.g., `docker run -e DB_HOST=localhost mysql`).
- **Debugging**: Use `docker logs`, `docker exec -it <container> /bin/bash` to troubleshoot.

**AWS Context**: In AWS EKS, containers run on EC2 instances or Fargate, managed by Kubernetes. Docker images are stored in Amazon ECR.

### 2. Kubernetes Fundamentals
**Overview**: Kubernetes orchestrates containers at scale, managing deployments, scaling, and networking. Pods are the smallest deployable units, containing one or more containers. Services and Ingresses expose applications.

**How Achieved**:
- **Managing Containers at Scale**: Kubernetes schedules containers across nodes, ensuring high availability and load balancing.
- **Container Orchestrators**: Kubernetes won over Docker Swarm and Mesos due to its robust ecosystem and community support.
- **Scheduler as Tetris Player**: The Kubernetes scheduler assigns Pods to nodes based on resource availability, affinity rules, and constraints.
- **Kubernetes as IaaS**: Provides abstractions like Pods, Services, and Ingresses to manage infrastructure declaratively.
- **Pods, Services, Ingresses**: Pods run containers; Services provide stable networking; Ingresses handle external HTTP routing.
- **Local Cluster with Minikube**: Use `minikube start` to create a single-node cluster for testing.
- **Creating Deployments**: Use `kubectl apply -f deployment.yaml` to define replicas and container specs.
- **Exposing Deployments**: Create Services with `kubectl expose deployment myapp --type=ClusterIP --port=80`.
- **Scaling**: Adjust replicas in Deployment (e.g., `kubectl scale deployment myapp --replicas=3`).
- **Testing Resiliency**: Simulate node failures with `kubectl delete node` or pod evictions to verify recovery.

**AWS Context**: EKS manages Kubernetes control plane, with worker nodes on EC2 or Fargate. Use `eksctl` to create clusters.

### 3. Deployment Strategies
**Overview**: Kubernetes supports zero-downtime deployments using rolling updates, canary deployments, and blue-green deployments to ensure smooth application updates.

**How Achieved**:
- **Monitoring Uptime**: Use liveness, readiness, and startup probes to check application health.
- **Probes**: Liveness (`livenessProbe`) restarts unhealthy Pods; Readiness (`readinessProbe`) controls traffic; Startup (`startupProbe`) delays checks for slow apps. Defined in Pod spec (e.g., HTTP or TCP checks).
- **Rolling Updates**: Set `strategy: rollingUpdate` in Deployment to update Pods incrementally (e.g., `maxSurge=25%`, `maxUnavailable=25%`).
- **Labels and Selectors**: Use labels (e.g., `app: myapp`) and selectors to target Pods in Services and Deployments.
- **Canary Deployments**: Deploy a new version to a subset of Pods, monitor, then scale (e.g., separate Deployment with 10% replicas).
- **Blue-Green Deployments**: Run two environments (blue: old, green: new), switch traffic via Service selector.
- **Rollbacks**: Use `kubectl rollout undo deployment/myapp` to revert to previous version.

**AWS Context**: In EKS, use ALB Ingress Controller for traffic routing in canary or blue-green deployments.

## Day 2: Templating and Kubernetes Architecture

### 1. Templating Kubernetes Resources
**Overview**: Templating simplifies managing Kubernetes manifests across environments using tools like Helm and Kustomize.

**How Achieved**:
- **Reusable Templates**: Define parameterized YAMLs to avoid duplication.
- **Helm**: Uses Go-based templating with `values.yaml` for customization. Install charts with `helm install myapp ./chart`.
- **Helm Architecture**: Comprises Helm client and Tiller (pre-v3) or direct API interaction (v3+).
- **Go and Sprig**: Helm uses Go templates with Sprig functions for logic (e.g., `{{ .Values.replicaCount }}`).
- **Managing Releases**: Use `helm upgrade` or `helm rollback` for updates and reversions.
- **Kustomize**: Applies patches to base YAMLs (e.g., `kustomization.yaml` for environment-specific overrides).
- **Repositories**: Store Helm charts in repositories like ChartMuseum or AWS S3.
- **Kubernetes SDKs**: Use client-go or Python SDK for programmatic templating.

**AWS Context**: Store Helm charts in Amazon S3 or ECR for EKS deployments.

### 2. Kubernetes Architecture
**Overview**: Kubernetes comprises a control plane (API server, etcd, scheduler, controller manager) and worker nodes (kubelet, kube-proxy).

**How Achieved**:
- **Single/Multi-Node Clusters**: Use `kubeadm init` for single-node or multi-node setups.
- **Control Plane**: API server handles requests, etcd stores state, scheduler places Pods, controller manager manages replication.
- **etcd and RAFT**: etcd uses RAFT consensus for distributed state consistency.
- **Kubelet**: Runs on nodes, manages Pods, and reports to the control plane.
- **Multi-Master**: Set up HA with multiple API servers and etcd instances.
- **Kubeadm**: Bootstrap clusters with `kubeadm init` and `kubeadm join`.
- **Overlay Network**: Install CNI plugins like Calico for pod networking.
- **Ingress Controller**: Deploy NGINX or ALB Ingress for external traffic.
- **API Exploration**: Use `curl` or client-go to interact with Kubernetes API directly.

**AWS Context**: EKS manages control plane; worker nodes use CNI plugins like AWS VPC CNI.

## Day 3: Networking and Advanced Scheduling

### 1. Cluster Networking and East-West Traffic
**Overview**: Kubernetes manages internal pod-to-pod communication using CNI plugins and Services.

**How Achieved**:
- **Network Requirements**: Pods get unique IPs; CNI plugins (e.g., Calico, Flannel) handle routing.
- **Service Discovery**: Services provide stable IPs for Pods using DNS (CoreDNS).
- **Kube-Proxy**: Routes traffic to Pods using iptables or IPVS.
- **Service Types**: ClusterIP for internal, Headless for direct pod access.
- **Network Policies**: Define rules to control pod communication (e.g., allow specific namespaces).

**AWS Context**: AWS VPC CNI assigns VPC IPs to Pods in EKS.

### 2. North-South Traffic and Service Meshes
**Overview**: Expose applications externally using NodePort, LoadBalancer, or Ingress. Service meshes enhance traffic management.

**How Achieved**:
- **NodePort**: Exposes Pods on node ports (30000-32767).
- **LoadBalancer**: Provisions cloud load balancers (e.g., AWS ELB).
- **Ingress**: Routes HTTP traffic based on domains/paths using controllers like NGINX or AWS ALB.
- **Service Mesh (Istio)**: Adds observability, security, and traffic control (e.g., circuit breaking).

**AWS Context**: Use AWS ALB Ingress Controller for EKS.

### 3. Advanced Scheduling
**Overview**: Control Pod placement using node selectors, affinity, taints, and tolerations.

**How Achieved**:
- **Scheduler**: Uses filters (resource availability) and predicates (affinity rules) to place Pods.
- **NodeSelector**: Assign Pods to nodes with specific labels (e.g., `nodeSelector: disktype=ssd`).
- **Affinity/Anti-Affinity**: Define Pod or node affinity rules (e.g., co-locate Pods).
- **Taints/Tolerations**: Restrict Pods from nodes unless tolerated (e.g., `tolerations: key=dedicated`).
- **Topology Spread**: Distribute Pods across zones for HA.

**AWS Context**: Use node affinity in EKS to schedule on specific instance types (e.g., GPU instances).

## Day 4: Managing State, Autoscaling, and Security

### 1. Managing State in Kubernetes
**Overview**: Kubernetes handles stateful applications using Persistent Volumes (PVs) and StatefulSets.

**How Achieved**:
- **Configs and Secrets**: Use ConfigMaps and Secrets for configuration data.
- **Volumes**: Attach storage to Pods (e.g., `emptyDir`, `hostPath`).
- **PV/PVC**: Define PVs for storage backends (e.g., AWS EBS) and PVCs for requests.
- **StatefulSets**: Deploy stateful apps like databases with stable identities.
- **Dynamic Provisioning**: Use StorageClasses for automatic volume creation.

**AWS Context**: Use EBS or EFS with StorageClasses in EKS.

### 2. Autoscaling
**Overview**: Automatically scale Pods or nodes based on metrics.

**How Achieved**:
- **HPA**: Horizontal Pod Autoscaler scales Pods based on CPU/memory or custom metrics.
- **Metrics Server**: Collects resource usage for HPA.
- **Prometheus**: Expose custom metrics for scaling (e.g., requests per second).
- **Cluster Autoscaler**: Scales worker nodes based on demand.

**AWS Context**: Use AWS Autoscaling Groups with Cluster Autoscaler in EKS.

### 3. Security
**Overview**: Secure Kubernetes clusters against threats using RBAC, network policies, and container security.

**How Achieved**:
- **Threat Matrix**: Address risks like unauthorized access or container escapes.
- **Docker Security**: Use non-root users, minimal images.
- **Kubernetes Security**: Enable RBAC, use namespaces, and secure etcd.
- **Network Security**: Apply NetworkPolicies to restrict traffic.
- **Authentication**: Use OIDC or certificates for user authentication.

**AWS Context**: Integrate EKS with IAM for RBAC.

## Additional Topics

### Service Meshes (Istio)
**Overview**: Istio provides advanced traffic management, security, and observability.
**How Achieved**: Deploy Istio sidecars to Pods for mTLS, traffic routing, and metrics.

### CKA/CKAD Exam Prep
**Overview**: Focus on practical tasks like debugging Pods, creating Deployments, and configuring RBAC.
**How Achieved**: Practice with `kubectl`, understand YAML manifests, and simulate cluster failures.

### Logging and Monitoring
**Overview**: Use tools like Prometheus, Grafana, and EFK for observability.
**How Achieved**: Deploy Prometheus for metrics, Grafana for visualization, and Fluentd for logs.

**AWS Context**: Use CloudWatch for logging and monitoring in EKS.

## Interview Questions and Answers

### Linux Containers and Docker
1. **What are the key differences between containers and VMs? (Beginner)**
   - **Answer**: Containers share the host OS kernel, using namespaces and cgroups for isolation, making them lightweight and fast. VMs include a full OS, requiring more resources. Containers start faster (seconds vs. minutes) and use less disk space (MBs vs. GBs). In AWS, containers run in ECS/EKS, while VMs are EC2 instances.
   - **Practical**: Demonstrate with `docker run nginx` vs. launching an EC2 instance.

2. **How does Docker’s client-server architecture work? (Intermediate)**
   - **Answer**: Docker uses a client-server model where the Docker CLI sends commands to the Docker daemon (dockerd) via a REST API over UNIX sockets or TCP. The daemon manages containers, images, and volumes, interacting with the host OS. Example: `docker run` sends a request to dockerd to pull an image and start a container.
   - **Practical**: Show `docker info` to inspect daemon status.

3. **How would you debug a container that’s crashing repeatedly? (Advanced)**
   - **Answer**: Check logs with `docker logs <container>`, inspect configuration with `docker inspect`, and access the container with `docker exec -it <container> /bin/bash` to check processes or files. Verify resource limits (CPU/memory) and environment variables. In EKS, use `kubectl logs` and `kubectl describe pod`.
   - **Practical**: Simulate a crash by setting invalid `CMD` in a Dockerfile and debug.

### Kubernetes Fundamentals
4. **What is a Pod, and how does it differ from a container? (Beginner)**
   - **Answer**: A Pod is the smallest unit in Kubernetes, containing one or more containers that share network (localhost) and storage (volumes). Containers in a Pod are co-located and managed together. A single-container Pod is common, but multi-container Pods are used for sidecars (e.g., logging). In EKS, Pods run on EC2 or Fargate.
   - **Practical**: Create a Pod with `kubectl apply -f pod.yaml`.

5. **How does the Kubernetes scheduler work? (Intermediate)**
   - **Answer**: The scheduler assigns Pods to nodes based on filters (resource availability, taints) and predicates (affinity, constraints). It scores nodes and selects the best fit. Example: A Pod with `nodeSelector: disktype=ssd` is scheduled on matching nodes. In EKS, the scheduler works with EC2 instance types.
   - **Practical**: Use `kubectl describe pod` to view scheduling events.

6. **How would you scale an application in Kubernetes and ensure it’s resilient? (Advanced)**
   - **Answer**: Scale using `kubectl scale deployment myapp --replicas=5` or HPA with custom metrics (e.g., Prometheus). Ensure resiliency with liveness/readiness probes, multiple replicas, and PodDisruptionBudgets. Test by simulating node failures with `kubectl drain`. In EKS, use Cluster Autoscaler for node scaling.
   - **Practical**: Deploy an HPA with `kubectl autoscale deployment myapp --min=2 --max=10 --cpu-percent=80`.

### Deployment Strategies
7. **What are the differences between rolling updates, canary, and blue-green deployments? (Beginner)**
   - **Answer**: Rolling updates incrementally replace Pods with new versions, minimizing downtime. Canary deployments release a new version to a small subset of users for testing. Blue-green deployments run old and new versions in parallel, switching traffic instantly. In EKS, use ALB for canary traffic splitting.
   - **Practical**: Configure `rollingUpdate` in a Deployment YAML.

8. **How do you implement a canary deployment in Kubernetes? (Intermediate)**
   - **Answer**: Create a separate Deployment for the canary version with fewer replicas (e.g., 10%). Use a Service with selectors to route a portion of traffic. Monitor with Prometheus/Grafana, then scale up the canary or rollback. Alternatively, use Istio for traffic splitting. In EKS, configure ALB weights for canary routing.
   - **Practical**: Write a canary Deployment YAML and Service.

9. **How would you achieve zero-downtime deployments in Kubernetes? (Advanced)**
   - **Answer**: Use rolling updates with `maxSurge` and `maxUnavailable` to ensure new Pods are ready before old ones are terminated. Configure readiness probes to verify app health. Use PodDisruptionBudgets to maintain availability during updates. In EKS, test with `kubectl apply` and monitor with CloudWatch.
   - **Practical**: Deploy a rolling update with `kubectl apply -f deployment.yaml`.

### Templating Kubernetes Resources
10. **What is Helm, and how does it simplify Kubernetes deployments? (Beginner)**
    - **Answer**: Helm is a package manager for Kubernetes, using charts (templates + values) to define resources. It simplifies deployments by parameterizing YAMLs, enabling reuse across environments. Example: `helm install myapp ./chart` deploys a preconfigured app.
    - **Practical**: Install a chart with `helm install`.

11. **How does Kustomize differ from Helm for templating? (Intermediate)**
    - **Answer**: Kustomize applies patches to base YAMLs without templating, maintaining native Kubernetes manifests. Helm uses Go templates for dynamic generation. Kustomize is simpler for static changes, while Helm is better for complex parameterization. In EKS, use Kustomize for environment-specific overlays.
    - **Practical**: Create a `kustomization.yaml` for a dev environment.

12. **How would you manage Helm chart dependencies across teams? (Advanced)**
    - **Answer**: Store charts in a central repository (e.g., ChartMuseum, AWS S3). Define dependencies in `Chart.yaml` and update with `helm dependency update`. Use version pinning for stability. Implement CI/CD pipelines (e.g., Jenkins) to test and deploy charts. In EKS, integrate with ECR for chart storage.
    - **Practical**: Configure a chart with dependencies and deploy.

### Kubernetes Architecture
13. **What are the core components of the Kubernetes control plane? (Beginner)**
    - **Answer**: The control plane includes the API server (handles requests), etcd (stores state), scheduler (places Pods), and controller manager (manages replication). In EKS, AWS manages the control plane.
    - **Practical**: Use `kubectl get componentstatuses` to check control plane health.

14. **How does etcd ensure data consistency in Kubernetes? (Intermediate)**
    - **Answer**: etcd uses the RAFT consensus algorithm to ensure distributed consistency across replicas. It stores cluster state as key-value pairs, syncing changes via leader election. In EKS, etcd is managed by AWS.
    - **Practical**: Backup etcd with `etcdctl snapshot save`.

15. **How would you set up a highly available Kubernetes cluster? (Advanced)**
    - **Answer**: Use multiple control plane nodes with load-balanced API servers, replicated etcd instances, and multi-node workers. Configure with `kubeadm init --control-plane-endpoint`. Ensure no single point of failure by spreading nodes across AZs. In EKS, enable multi-AZ control plane and use Cluster Autoscaler.
    - **Practical**: Deploy a multi-master cluster with `kubeadm`.

### Cluster Networking and East-West Traffic
16. **How does Kubernetes handle pod-to-pod communication? (Beginner)**
    - **Answer**: Pods communicate via ClusterIP Services, using CNI plugins (e.g., Calico) for networking. Each Pod gets a unique IP, and CoreDNS resolves Service names. Kube-proxy routes traffic using iptables or IPVS.
    - **Practical**: Create a ClusterIP Service with `kubectl expose`.

17. **What are Network Policies, and how do they secure east-west traffic? (Intermediate)**
    - **Answer**: Network Policies define rules to control pod communication (e.g., allow traffic from specific namespaces). They use labels to target Pods and specify ingress/egress rules. Requires a CNI plugin like Calico. In EKS, apply policies to secure microservices.
    - **Practical**: Write a NetworkPolicy YAML to restrict traffic.

18. **How would you troubleshoot network issues in a Kubernetes cluster? (Advanced)**
    - **Answer**: Check Pod logs (`kubectl logs`), verify Service endpoints (`kubectl get endpoints`), and inspect Network Policies. Use `kubectl describe pod` to check DNS or CNI issues. Test connectivity with `kubectl exec` and `ping`. In EKS, verify VPC CNI configuration and security groups.
    - **Practical**: Simulate a network issue and debug.

### North-South Traffic and Service Meshes
19. **What’s the difference between NodePort and LoadBalancer Service types? (Beginner)**
    - **Answer**: NodePort exposes a Service on a node’s port (30000-32767), accessible externally. LoadBalancer provisions a cloud load balancer (e.g., AWS ELB) for external access. NodePort is simpler but less scalable; LoadBalancer is production-ready.
    - **Practical**: Create a LoadBalancer Service in EKS.

20. **How does an Ingress controller work in Kubernetes? (Intermediate)**
    - **Answer**: Ingress controllers (e.g., NGINX, AWS ALB) route HTTP/HTTPS traffic based on Ingress rules (domain/path). They provide load balancing, SSL termination, and path-based routing. In EKS, use AWS ALB Ingress Controller.
    - **Practical**: Deploy an Ingress with `kubectl apply -f ingress.yaml`.

21. **How does a service mesh like Istio enhance Kubernetes networking? (Advanced)**
    - **Answer**: Istio injects Envoy sidecars to Pods, providing mTLS, traffic routing, observability (Prometheus metrics), and fault tolerance (circuit breaking). It simplifies canary deployments and A/B testing. In EKS, deploy Istio with `istioctl install`.
    - **Practical**: Configure Istio for traffic splitting.

### Advanced Scheduling
22. **What is node affinity, and how does it differ from nodeSelector? (Beginner)**
    - **Answer**: Node affinity defines rules for Pod placement (e.g., prefer nodes with `disktype=ssd`), offering more flexibility than nodeSelector’s strict key-value matching. Affinity supports required and preferred rules.
    - **Practical**: Add affinity to a Pod spec.

23. **How do taints and tolerations work in Kubernetes? (Intermediate)**
    - **Answer**: Taints mark nodes to repel Pods unless they have matching tolerations. Example: `kubectl taint nodes node1 key=value:NoSchedule` prevents scheduling unless a Pod has `tolerations: key=value`. Useful for dedicated nodes.
    - **Practical**: Apply a taint and toleration.

24. **How would you schedule machine learning workloads to GPU nodes? (Advanced)**
    - **Answer**: Use node affinity to target GPU nodes (e.g., `nodeAffinity: requiredDuringSchedulingIgnoredDuringExecution`). Add taints to GPU nodes and tolerations to ML Pods. Use NVIDIA device plugins for GPU access. In EKS, deploy on GPU instance types (e.g., g4dn).
    - **Practical**: Write a Pod spec for GPU scheduling.

### Managing State in Kubernetes
25. **What are Persistent Volumes and Persistent Volume Claims? (Beginner)**
    - **Answer**: PVs define storage resources (e.g., AWS EBS). PVCs are requests for storage by Pods, bound to PVs dynamically or statically. StorageClasses enable dynamic provisioning.
    - **Practical**: Create a PVC and StorageClass.

26. **How do you deploy a stateful application like a database in Kubernetes? (Intermediate)**
    - **Answer**: Use StatefulSets for stable Pod identities and persistent storage. Define PVCs for each Pod and a headless Service for DNS. Example: Deploy MySQL with a StatefulSet and EBS volumes in EKS.
    - **Practical**: Write a StatefulSet YAML for MySQL.

27. **How would you ensure data persistence in a clustered database? (Advanced)**
    - **Answer**: Use StatefulSets with PVCs for each replica, backed by replicated storage (e.g., AWS EFS). Configure replication in the database (e.g., MySQL Galera). Use readiness probes to ensure availability. In EKS, use EFS CSI driver.
    - **Practical**: Deploy a clustered database with persistence.

### Autoscaling
28. **What is the Horizontal Pod Autoscaler, and how does it work? (Beginner)**
    - **Answer**: HPA scales Pods based on metrics like CPU/memory usage or custom metrics. It requires Metrics Server or Prometheus. Example: `kubectl autoscale deployment myapp --min=2 --max=10 --cpu-percent=80`.
    - **Practical**: Deploy an HPA.

29. **How do you scale based on custom metrics? (Intermediate)**
    - **Answer**: Use Prometheus to expose custom metrics (e.g., HTTP requests). Configure a custom metrics adapter (e.g., Prometheus Adapter) to feed metrics to HPA. Define HPA with `metrics: type=Pods`.
    - **Practical**: Set up HPA with custom metrics.

30. **How would you implement cluster autoscaling in a production environment? (Advanced)**
    - **Answer**: Deploy Cluster Autoscaler to scale worker nodes based on Pod demand. Configure with Autoscaling Groups in EKS. Set node group min/max sizes and monitor with CloudWatch. Test by increasing Pod replicas to trigger scaling.
    - **Practical**: Deploy Cluster Autoscaler in EKS.

### Security
31. **What are the key security best practices for Kubernetes clusters? (Beginner)**
    - **Answer**: Enable RBAC, use namespaces, apply Network Policies, run containers as non-root, and secure etcd. Regularly update Kubernetes and scan images for vulnerabilities.
    - **Practical**: Create an RBAC role.

32. **How does RBAC work in Kubernetes? (Intermediate)**
    - **Answer**: RBAC controls access using Roles/ClusterRoles and RoleBindings/ClusterRoleBindings. Example: Grant `pod-reader` role to a user for `get` and `list` verbs on Pods. In EKS, map IAM roles to RBAC.
    - **Practical**: Write an RBAC Role and RoleBinding.

33. **How would you secure a Kubernetes cluster against a malicious attack? (Advanced)**
    - **Answer**: Implement RBAC, Network Policies, and PodSecurityPolicies (or Pod Security Admission). Use mTLS with Istio, scan images with Trivy, and monitor with Falco. In EKS, use IAM roles and security groups. Simulate attacks to test defenses.
    - **Practical**: Deploy a Network Policy and test.

### Additional Topics
34. **How does Istio enhance Kubernetes deployments? (Intermediate)**
    - **Answer**: Istio provides mTLS, traffic routing, observability, and fault tolerance via Envoy sidecars. It supports canary deployments, circuit breaking, and metrics collection. In EKS, deploy Istio for microservices.
    - **Practical**: Install Istio and configure a VirtualService.

35. **What are key tasks for CKA/CKAD exam preparation? (Intermediate)**
    - **Answer**: Practice creating Deployments, debugging Pods, configuring RBAC, and managing storage. Use `kubectl` proficiently and understand YAML manifests. Simulate cluster failures for troubleshooting.
    - **Practical**: Debug a failed Pod with `kubectl describe`.

36. **How do you implement logging and monitoring in Kubernetes? (Advanced)**
    - **Answer**: Deploy EFK (Elasticsearch, Fluentd, Kibana) for logging and Prometheus/Grafana for monitoring. Configure Fluentd to collect Pod logs and Prometheus to scrape metrics. In EKS, use CloudWatch for centralized logging.
    - **Practical**: Deploy Prometheus and Grafana.

## Tips for Interview Success
- **Explain Concepts Clearly**: Use analogies (e.g., scheduler as Tetris) to simplify complex topics.
- **Relate to AWS**: Mention EKS, ECR, ALB, and CloudWatch where applicable.
- **Show Hands-On Experience**: Reference commands like `kubectl apply`, `docker build`, or `helm install`.
- **Prepare for Scenarios**: Be ready to troubleshoot (e.g., Pod not starting, network issues).
- **Behavioral Questions**: Highlight teamwork, problem-solving, and your Movate projects (e.g., CI/CD pipelines with Jenkins).
