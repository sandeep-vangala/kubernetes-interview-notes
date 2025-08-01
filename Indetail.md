# Detailed Kubernetes Interview Preparation Notes


## Day 1: Linux Containers, Kubernetes Fundamentals, and Deployment Strategies

### 1. Linux Containers and Kubernetes

#### 1.1 Containers vs VMs
- **Why**: Containers are lightweight, enabling faster deployment, portability, and efficient resource use compared to VMs, which is critical for modern microservices and Kubernetes’ container orchestration model. Understanding this distinction showcases your grasp of containerization’s role in DevOps.
- **What**: Containers share the host OS kernel, using namespaces (PID, network, mount) and cgroups for isolation, consuming MBs of storage and starting in seconds. VMs run a full OS with a hypervisor, requiring GBs of storage and minutes to boot. Containers are ideal for Kubernetes Pods, while VMs are used for traditional monolithic apps or isolation-heavy workloads.
- **How**: Containers are created using runtimes like Docker or containerd. Example: `docker run -d nginx` starts a container sharing the host kernel. VMs are managed via hypervisors like VMware or AWS EC2. In EKS, containers run on EC2 worker nodes or Fargate, abstracting VM management.
- **AWS Context**: EKS uses containers for Pods, stored in Amazon ECR, while EC2 instances or Fargate provide the underlying compute.
- **Interview Relevance**: Expect questions like “Why use containers over VMs in Kubernetes?” Answer by emphasizing efficiency and portability, referencing EKS deployments.

#### 1.2 Understanding Process Isolation
- **Why**: Process isolation ensures containers run independently, preventing interference, which is foundational for secure and reliable Kubernetes workloads. This knowledge demonstrates your understanding of container security.
- **What**: Linux namespaces (PID, network, mount, user) isolate processes, giving each container its own process tree, network stack, and filesystem. Cgroups limit resources (CPU, memory, I/O), ensuring fair usage.
- **How**: Docker configures namespaces and cgroups automatically when running containers. Example: `docker run --cpus=0.5 --memory=512m nginx` limits resources. Inspect with `docker inspect <container>` to view namespace details. In Kubernetes, the kubelet manages these settings for Pods.
- **AWS Context**: EKS Pods inherit isolation from the container runtime (e.g., containerd). Use AWS Security Groups for additional network isolation.
- **Interview Relevance**: Questions like “How does Kubernetes ensure container isolation?” require explaining namespaces/cgroups, tying to EKS security.

#### 1.3 Is Docker the Only One?
- **Why**: Knowing container runtime alternatives highlights your awareness of Kubernetes’ flexibility, as it supports multiple runtimes, adapting to enterprise needs.
- **What**: Docker was the default runtime but has been replaced by containerd or CRI-O in modern Kubernetes due to Docker’s monolithic design. Containerd is lightweight, focusing on container execution, while CRI-O is Kubernetes-specific.
- **How**: Kubernetes uses the Container Runtime Interface (CRI) to interact with runtimes. Check the runtime with `kubectl get nodes -o wide` (shows containerd or Docker). Switch runtimes in EKS by configuring the node group’s bootstrap script.
- **AWS Context**: EKS defaults to containerd since Kubernetes 1.24. Use custom AMIs to configure CRI-O if needed.
- **Interview Relevance**: Be ready for “What container runtimes does Kubernetes support?” Mention Docker, containerd, CRI-O, and EKS defaults.

#### 1.4 Docker Client-Server Architecture
- **Why**: Understanding Docker’s architecture is key for troubleshooting and optimizing container workflows, a common DevOps task in Kubernetes environments.
- **What**: Docker CLI (client) sends commands to the Docker daemon (dockerd) via a REST API over UNIX sockets or TCP. Dockerd manages images, containers, networks, and volumes, interacting with the OS.
- **How**: Run `docker run -d -p 80:80 nginx` to see the CLI communicate with dockerd. Inspect daemon logs with `journalctl -u docker.service`. In Kubernetes, the kubelet interacts with the runtime (e.g., containerd) instead of dockerd directly.
- **AWS Context**: In EKS, containerd replaces dockerd on worker nodes, but you may encounter Docker in CI/CD pipelines (e.g., Jenkins building images).
- **Interview Relevance**: Questions like “How does Docker manage containers?” require explaining client-server communication, referencing EKS runtimes.

#### 1.5 Running Containers
- **Why**: Running containers is a core DevOps skill, enabling application deployment and testing, directly applicable to Kubernetes Pods.
- **What**: Use `docker run` to start containers from images, specifying options like ports, volumes, and environment variables.
- **How**: Example: `docker run -d --name myapp -p 8080:80 nginx` starts an NGINX container. Verify with `docker ps`. In Kubernetes, Pods define containers via YAML (e.g., `spec.containers.image=nginx`).
- **AWS Context**: Push images to ECR (`aws ecr get-login-password | docker login`) and run in EKS Pods.
- **Interview Relevance**: Expect “How do you run a container in Docker?” Demonstrate with `docker run` and relate to Kubernetes Pods.

#### 1.6 Building Docker Images
- **Why**: Building custom images is essential for packaging applications, ensuring consistency across environments, a key CI/CD task.
- **What**: Dockerfiles define image layers (e.g., `FROM`, `COPY`, `RUN`). Images are cached for efficiency.
- **How**: Create a Dockerfile: `FROM node:16`, `COPY . /app`, `RUN npm install`, `CMD ["npm", "start"]`. Build with `docker build -t myapp:1.0 .`. Push to ECR with `docker push`.
- **AWS Context**: Use Jenkins or CodeBuild to automate image builds, storing in ECR for EKS deployments.
- **Interview Relevance**: Questions like “How do you build a Docker image?” require a Dockerfile example and CI/CD integration.

#### 1.7 Mounting Volumes
- **Why**: Volumes persist data beyond container lifecycles, critical for stateful applications like databases in Kubernetes.
- **What**: Volumes map host or external storage to container paths, preserving data.
- **How**: Use `docker run -v /host:/container nginx` for host mounts or `-v myvolume:/data` for named volumes. In Kubernetes, use PersistentVolumeClaims (PVCs) for volumes.
- **AWS Context**: In EKS, use EBS or EFS volumes via PVCs.
- **Interview Relevance**: Be ready for “How do you persist data in containers?” Explain Docker volumes and Kubernetes PVCs.

#### 1.8 Exposing Ports
- **Why**: Exposing ports enables container communication, crucial for microservices and Kubernetes Services.
- **What**: Map container ports to host ports to allow external access.
- **How**: Use `docker run -p 8080:80 nginx` to map port 80 (container) to 8080 (host). In Kubernetes, Services expose Pod ports (e.g., `port: 80`).
- **AWS Context**: EKS Services use AWS ALB/ELB for external access.
- **Interview Relevance**: Questions like “How do you expose a container port?” require `docker run -p` and Kubernetes Service examples.

#### 1.9 Managing Containers Lifecycle
- **Why**: Lifecycle management ensures efficient resource use and application availability, a DevOps responsibility.
- **What**: Includes starting, stopping, restarting, and removing containers.
- **How**: Use `docker start`, `stop`, `restart`, `rm`. Monitor with `docker ps`. In Kubernetes, the kubelet manages Pod lifecycles.
- **AWS Context**: EKS automates lifecycle via Deployments/StatefulSets.
- **Interview Relevance**: Expect “How do you manage container lifecycles?” Mention Docker commands and Kubernetes controllers.

#### 1.10 Injecting Environment Variables
- **Why**: Environment variables configure applications dynamically, enabling flexibility across environments.
- **What**: Pass key-value pairs to containers at runtime.
- **How**: Use `docker run -e DB_HOST=localhost nginx`. In Kubernetes, define `env` in Pod specs (e.g., `env: name=DB_HOST, value=localhost`).
- **AWS Context**: Use AWS Secrets Manager or ConfigMaps in EKS for sensitive variables.
- **Interview Relevance**: Questions like “How do you configure containers?” Include `docker run -e` and Kubernetes `env`.

#### 1.11 Debugging Running Containers
- **Why**: Debugging skills are critical for resolving production issues, a key DevOps competency.
- **What**: Inspect logs, access containers, and analyze configurations.
- **How**: Use `docker logs <container>`, `docker exec -it <container> /bin/bash`, and `docker inspect`. In Kubernetes, use `kubectl logs`, `exec`, and `describe pod`.
- **AWS Context**: In EKS, integrate CloudWatch for centralized logging.
- **Interview Relevance**: Be ready for “How do you debug a crashing container?” Demonstrate Docker/Kubernetes commands.

### 2. Kubernetes Fundamentals

#### 2.1 Managing Containers at Scale
- **Why**: Scaling is essential for high availability and performance, a core Kubernetes feature.
- **What**: Kubernetes orchestrates thousands of containers across nodes, balancing load and ensuring resiliency.
- **How**: Use Deployments to manage replicas (`replicas: 3`) and Services for load balancing. Scale with `kubectl scale deployment myapp --replicas=5`.
- **AWS Context**: EKS scales Pods on EC2/Fargate, with Cluster Autoscaler for nodes.
- **Interview Relevance**: Questions like “How does Kubernetes scale apps?” Require explaining Deployments and EKS scaling.

#### 2.2 The Battle of Container Orchestrators
- **Why**: Understanding Kubernetes’ dominance provides context for its adoption, showing industry awareness.
- **What**: Kubernetes outcompeted Docker Swarm and Mesos due to its robust ecosystem, community, and features like self-healing.
- **How**: Kubernetes’ declarative API and controllers (e.g., ReplicaSet) ensure consistent state. Deploy with `kubectl apply -f deployment.yaml`.
- **AWS Context**: EKS is AWS’s managed Kubernetes, preferred over ECS for complex workloads.
- **Interview Relevance**: Expect “Why Kubernetes over other orchestrators?” Highlight ecosystem and EKS adoption.

#### 2.3 Visualising the Data Centre as a Single VM
- **Why**: This abstraction simplifies infrastructure management, a key Kubernetes benefit.
- **What**: Kubernetes treats the cluster as a unified compute resource, abstracting node details.
- **How**: The scheduler assigns Pods to nodes based on resources, using `kubectl get nodes` to view the cluster. Users focus on application logic, not hardware.
- **AWS Context**: EKS abstracts EC2/Fargate nodes into a single cluster.
- **Interview Relevance**: Questions like “How does Kubernetes abstract infrastructure?” Use this analogy.

#### 2.4 The Scheduler is the Best Tetris Player
- **Why**: The scheduler ensures optimal resource utilization, critical for cost-efficiency and performance.
- **What**: The scheduler places Pods on nodes based on resource requests, affinity, and constraints.
- **How**: Define resource requests/limits in Pod specs (e.g., `resources.requests.cpu=500m`). Use `kubectl describe pod` to view scheduling decisions.
- **AWS Context**: EKS scheduler optimizes EC2/Fargate placement.
- **Interview Relevance**: Be ready for “How does the Kubernetes scheduler work?” Explain filters/predicates.

#### 2.5 Kubernetes as an Infrastructure as a Service Provider
- **Why**: Kubernetes’ IaaS-like abstractions simplify DevOps workflows, a key interview topic.
- **What**: Provides Pods, Services, Ingresses, and storage as managed resources, similar to AWS services.
- **How**: Define resources declaratively (e.g., `kubectl apply -f service.yaml`). Kubernetes controllers maintain desired state.
- **AWS Context**: EKS integrates with AWS services (e.g., ALB for Ingress).
- **Interview Relevance**: Questions like “What abstractions does Kubernetes provide?” List Pods/Services/Ingress.

#### 2.6 What are Pods, Services, and Ingresses?
- **Why**: These are core Kubernetes constructs, essential for application deployment.
- **What**: Pods run containers, Services provide networking, Ingresses route external HTTP traffic.
- **How**: Create a Pod (`kubectl apply -f pod.yaml`), Service (`kubectl expose`), and Ingress (`kubectl apply -f ingress.yaml`).
- **AWS Context**: In EKS, use AWS ALB Ingress Controller.
- **Interview Relevance**: Expect “Explain Pods vs. Services.” Provide definitions and examples.

#### 2.7 Creating a Local Cluster with Minikube
- **Why**: Local clusters are useful for learning and testing, showcasing hands-on skills.
- **What**: Minikube runs a single-node Kubernetes cluster locally.
- **How**: Install Minikube, run `minikube start`, and use `kubectl` to manage. Example: `kubectl run nginx --image=nginx`.
- **AWS Context**: Use Minikube for local testing before deploying to EKS.
- **Interview Relevance**: Questions like “How do you set up a local Kubernetes cluster?” Mention Minikube.

#### 2.8 Creating a Deployment
- **Why**: Deployments ensure application availability and updates, a core DevOps task.
- **What**: Deployments manage ReplicaSets and Pods, ensuring desired replicas.
- **How**: Create with `kubectl apply -f deployment.yaml` (e.g., `replicas: 3, image: nginx`).
- **AWS Context**: Deploy to EKS with `eksctl` and `kubectl`.
- **Interview Relevance**: Be ready for “How do you create a Deployment?” Provide YAML example.

#### 2.9 Exposing Deployments
- **Why**: Exposing apps enables user access, a key microservices requirement.
- **What**: Services (e.g., ClusterIP, LoadBalancer) route traffic to Pods.
- **How**: Use `kubectl expose deployment myapp --type=LoadBalancer --port=80`. In EKS, this provisions an AWS ELB.
- **AWS Context**: Use ALB Ingress for advanced routing in EKS.
- **Interview Relevance**: Questions like “How do you expose a Kubernetes app?” Include Service types.

#### 2.10 What is a Pod?
- **Why**: Pods are the smallest unit in Kubernetes, foundational for understanding orchestration.
- **What**: A Pod runs one or more containers, sharing network/storage.
- **How**: Define in YAML (`spec.containers.image=nginx`) and apply with `kubectl apply`.
- **AWS Context**: EKS Pods run on EC2/Fargate.
- **Interview Relevance**: Expect “What is a Pod?” Explain with examples.

#### 2.11 Scaling Applications
- **Why**: Scaling ensures performance under load, a critical production skill.
- **What**: Adjust Pod replicas manually or via HPA.
- **How**: Use `kubectl scale deployment myapp --replicas=5` or deploy HPA (`kubectl autoscale`).
- **AWS Context**: EKS supports HPA with CloudWatch metrics.
- **Interview Relevance**: Be ready for “How do you scale in Kubernetes?” Mention HPA.

#### 2.12 Testing Resiliency
- **Why**: Resiliency ensures apps recover from failures, a DevOps priority.
- **What**: Simulate node/Pod failures to verify self-healing.
- **How**: Use `kubectl delete pod` or `kubectl drain node`. Deployments recreate Pods automatically.
- **AWS Context**: Test EKS resiliency with Cluster Autoscaler.
- **Interview Relevance**: Questions like “How do you test Kubernetes resiliency?” Describe simulation steps.

### 3. Deployment Strategies

#### 3.1 Monitoring for Uptime
- **Why**: Monitoring ensures application availability, critical for production.
- **What**: Use probes to check Pod health and metrics tools for performance.
- **How**: Define liveness/readiness probes in Pod specs. Monitor with Prometheus/Grafana.
- **AWS Context**: Use CloudWatch for EKS monitoring.
- **Interview Relevance**: Expect “How do you monitor Kubernetes apps?” Mention probes and tools.

#### 3.2 Liveness Probe
- **Why**: Liveness probes detect and restart unhealthy Pods, ensuring reliability.
- **What**: Checks if a container is running correctly (e.g., HTTP response).
- **How**: Define in YAML: `livenessProbe.httpGet.path=/health`. Kubernetes restarts failing Pods.
- **AWS Context**: Use in EKS to maintain app health.
- **Interview Relevance**: Questions like “What is a liveness probe?” Provide YAML example.

#### 3.3 Readiness Probe
- **Why**: Readiness probes prevent traffic to unready Pods, ensuring smooth updates.
- **What**: Checks if a Pod is ready to serve traffic.
- **How**: Define in YAML: `readinessProbe.httpGet.path=/ready`. Service excludes unready Pods.
- **AWS Context**: Critical for EKS rolling updates.
- **Interview Relevance**: Be ready for “How does a readiness probe work?” Explain traffic routing.

#### 3.4 Startup Probe
- **Why**: Startup probes handle slow-starting apps, preventing premature failures.
- **What**: Delays liveness checks until the app is fully started.
- **How**: Define in YAML: `startupProbe.tcpSocket.port=8080`. Used for legacy apps.
- **AWS Context**: Useful for EKS apps with long initialization.
- **Interview Relevance**: Questions like “When do you use a startup probe?” Describe use cases.

#### 3.5 Executing Zero Downtime Deployments with Rolling Updates
- **Why**: Zero-downtime deployments minimize user impact, a production requirement.
- **What**: Gradually replace old Pods with new ones.
- **How**: Set `strategy: rollingUpdate` in Deployment (e.g., `maxSurge=25%`). Apply with `kubectl apply`.
- **AWS Context**: Use with EKS and ALB for seamless updates.
- **Interview Relevance**: Expect “How do you achieve zero-downtime in Kubernetes?” Explain rolling updates.

#### 3.6 Using Labels and Selectors
- **Why**: Labels/selectors enable precise resource targeting, foundational for Services/Deployments.
- **What**: Key-value pairs (e.g., `app=myapp`) used to filter Pods.
- **How**: Define labels in Pod specs and selectors in Services (e.g., `selector: app=myapp`).
- **AWS Context**: Use in EKS for Service routing.
- **Interview Relevance**: Questions like “What are labels/selectors?” Provide examples.

#### 3.7 Testing Features with Canary Deployments
- **Why**: Canary deployments reduce risk by testing new features on a subset of users.
- **What**: Route a small percentage of traffic to a new version.
- **How**: Create a separate Deployment with fewer replicas, adjust Service weights (e.g., Istio). Monitor with Prometheus.
- **AWS Context**: Use ALB weights in EKS for canary routing.
- **Interview Relevance**: Be ready for “How do you implement canary deployments?” Describe steps.

#### 3.8 Releasing Features with Blue-Green Deployments
- **Why**: Blue-green deployments enable instant rollbacks, ideal for critical releases.
- **What**: Run old (blue) and new (green) versions, switching traffic instantly.
- **How**: Deploy green version, update Service selector to switch traffic. Rollback by reverting selector.
- **AWS Context**: Use EKS with ALB for traffic switching.
- **Interview Relevance**: Questions like “What is a blue-green deployment?” Explain with Kubernetes.

#### 3.9 Understanding Rollbacks and ReplicaSets
- **Why**: Rollbacks ensure quick recovery from failed updates, a production necessity.
- **What**: Revert to a previous Deployment version, managed by ReplicaSets.
- **How**: Use `kubectl rollout undo deployment/myapp`. Check history with `kubectl rollout history`.
- **AWS Context**: Test rollbacks in EKS for reliability.
- **Interview Relevance**: Expect “How do you rollback in Kubernetes?” Mention `kubectl rollout`.

## Day 2: Templating and Kubernetes Architecture

### 1. Templating Kubernetes Resources

#### 1.1 Creating Reusable Templates
- **Why**: Reusable templates reduce duplication, improving maintainability across environments.
- **What**: Parameterized YAMLs for resources like Deployments/Services.
- **How**: Use Helm or Kustomize to define templates. Example: Helm’s `values.yaml` customizes replicas.
- **AWS Context**: Use Helm charts in EKS for multi-environment deployments.
- **Interview Relevance**: Questions like “Why use templating in Kubernetes?” Highlight efficiency.

#### 1.2 Helm’s Templating Engine
- **Why**: Helm simplifies complex deployments, a common DevOps tool.
- **What**: Uses Go templates with Sprig functions for dynamic YAML generation.
- **How**: Define templates in `templates/deployment.yaml` (e.g., `{{ .Values.replicaCount }}`). Install with `helm install`.
- **AWS Context**: Store Helm charts in S3/ECR for EKS.
- **Interview Relevance**: Be ready for “How does Helm work?” Explain templates.

#### 1.3 Understanding the Helm Architecture
- **Why**: Knowing Helm’s architecture aids troubleshooting and optimization.
- **What**: Helm CLI interacts with Kubernetes API (v3+). Pre-v3 used Tiller.
- **How**: Run `helm install myapp ./chart` to apply resources. Debug with `helm template`.
- **AWS Context**: Use Helm with EKS RBAC for secure deployments.
- **Interview Relevance**: Questions like “What is Helm’s architecture?” Describe CLI-API interaction.

#### 1.4 Templating Resources with Go and Sprig
- **Why**: Go/Sprig enables advanced templating logic, critical for complex charts.
- **What**: Go templates with Sprig functions (e.g., `default`, `toYaml`).
- **How**: Example: `{{ .Values.image | default "nginx" }}` in a Helm template. Test with `helm template`.
- **AWS Context**: Use in EKS for dynamic configurations.
- **Interview Relevance**: Expect “How do you use Go in Helm?” Provide template examples.

#### 1.5 Managing Releases with Helm
- **Why**: Release management ensures controlled updates and rollbacks.
- **What**: Helm tracks releases, enabling upgrades/rollbacks.
- **How**: Use `helm upgrade myapp ./chart` for updates, `helm rollback myapp 1` for reversion.
- **AWS Context**: Integrate with EKS CI/CD pipelines.
- **Interview Relevance**: Questions like “How do you manage Helm releases?” Mention commands.

#### 1.6 Reverting Changes with Rollbacks
- **Why**: Rollbacks mitigate failed updates, ensuring stability.
- **What**: Revert to a previous Helm release.
- **How**: Use `helm rollback myapp 1`. Check history with `helm history myapp`.
- **AWS Context**: Test in EKS for production reliability.
- **Interview Relevance**: Be ready for “How do you rollback a Helm release?” Explain process.

#### 1.7 Depending on Other Charts
- **Why**: Dependencies enable modular charts, improving reuse.
- **What**: Charts can depend on others (e.g., MySQL chart for an app).
- **How**: Define in `Chart.yaml` (e.g., `dependencies: name=mysql`). Update with `helm dependency update`.
- **AWS Context**: Use in EKS for microservices.
- **Interview Relevance**: Questions like “How do Helm dependencies work?” Provide examples.

#### 1.8 Storing Reusable Templates in Repositories
- **Why**: Centralized repositories streamline chart sharing, a DevOps best practice.
- **What**: Store charts in ChartMuseum, S3, or public repos.
- **How**: Push charts with `helm push`. Add repo with `helm repo add`.
- **AWS Context**: Use S3 as a Helm repo for EKS.
- **Interview Relevance**: Expect “How do you share Helm charts?” Mention repositories.

#### 1.9 Structured Templating Introduction with YQ
- **Why**: YQ simplifies YAML manipulation, useful for custom templating.
- **What**: A command-line tool for querying/modifying YAML.
- **How**: Example: `yq eval '.replicas = 3' deployment.yaml`. Use in CI/CD pipelines.
- **AWS Context**: Integrate with EKS pipelines.
- **Interview Relevance**: Questions like “What is YQ?” Explain YAML manipulation.

#### 1.10 Kubernetes Specific Structured Templating with Kustomize
- **Why**: Kustomize offers native Kubernetes templating, avoiding external tools.
- **What**: Applies patches to base YAMLs for environment-specific configs.
- **How**: Create `kustomization.yaml` with bases/patches. Apply with `kubectl apply -k .`.
- **AWS Context**: Use Kustomize for EKS environment overlays.
- **Interview Relevance**: Be ready for “How does Kustomize differ from Helm?” Compare approaches.

#### 1.11 Using Kubernetes SDKs for Templating
- **Why**: SDKs enable programmatic resource management, useful for custom automation.
- **What**: Libraries like client-go or Python SDK interact with Kubernetes API.
- **How**: Write scripts to generate YAML (e.g., Python’s `kubernetes.client`). Example: Create a Deployment programmatically.
- **AWS Context**: Use SDKs in EKS for custom controllers.
- **Interview Relevance**: Questions like “How do you use Kubernetes SDKs?” Mention client-go.

### 2. Kubernetes Architecture

#### 2.1 Single and Multi-Node Clusters
- **Why**: Understanding cluster types informs scalability and HA strategies.
- **What**: Single-node clusters (e.g., Minikube) are for testing; multi-node for production.
- **How**: Set up with `kubeadm init` for single-node or `kubeadm join` for multi-node.
- **AWS Context**: EKS uses multi-node clusters across AZs.
- **Interview Relevance**: Expect “What’s a multi-node cluster?” Explain setup.

#### 2.2 Examining the Control Plane
- **Why**: The control plane is Kubernetes’ brain, critical for cluster management.
- **What**: Includes API server, etcd, scheduler, controller manager.
- **How**: Inspect with `kubectl get pods -n kube-system`. Check API server logs for troubleshooting.
- **AWS Context**: EKS manages control plane components.
- **Interview Relevance**: Questions like “What is the control plane?” List components.

#### 2.3 Persisting Changes in etcd
- **Why**: etcd ensures cluster state persistence, vital for reliability.
- **What**: A distributed key-value store using RAFT consensus.
- **How**: Backup with `etcdctl snapshot save`. Restore with `etcdctl snapshot restore`.
- **AWS Context**: EKS manages etcd, but backups are user responsibility.
- **Interview Relevance**: Be ready for “How does etcd work?” Explain RAFT.

#### 2.4 Syncing Changes with RAFT
- **Why**: RAFT ensures data consistency, critical for HA clusters.
- **What**: A consensus algorithm for leader election and replication.
- **How**: etcd nodes elect a leader, replicating writes. Monitor with `etcdctl endpoint status`.
- **AWS Context**: Managed by EKS, but understand for debugging.
- **Interview Relevance**: Questions like “What is RAFT in etcd?” Describe consensus.

#### 2.5 Event-Based Architecture
- **Why**: Event-driven design enables real-time state management, a Kubernetes strength.
- **What**: Components react to events (e.g., Pod creation) via API server.
- **How**: Watch events with `kubectl get events`. Controllers process events to reconcile state.
- **AWS Context**: EKS leverages events for autoscaling.
- **Interview Relevance**: Expect “How does Kubernetes handle events?” Explain architecture.

#### 2.6 Understanding the Kubelet
- **Why**: Kubelet manages node-level operations, essential for Pod execution.
- **What**: Runs on each node, interacting with the container runtime and API server.
- **How**: Check status with `systemctl status kubelet`. Configure via `/var/lib/kubelet/config.yaml`.
- **AWS Context**: EKS nodes run kubelet on EC2/Fargate.
- **Interview Relevance**: Questions like “What does the kubelet do?” Describe role.

#### 2.7 Verifying "No Single Point of Failure"
- **Why**: HA ensures cluster reliability, a production requirement.
- **What**: Multi-master setups eliminate single points of failure.
- **How**: Configure multiple API servers and etcd nodes with `kubeadm init --control-plane-endpoint`.
- **AWS Context**: EKS provides HA control plane by default.
- **Interview Relevance**: Be ready for “How do you ensure HA in Kubernetes?” Explain multi-master.

#### 2.8 Setting Up a Multi-Master Cluster
- **Why**: Multi-master clusters are standard for production, showcasing advanced skills.
- **What**: Multiple control plane nodes with load-balanced API access.
- **How**: Use `kubeadm init` with external load balancer, join masters with `kubeadm join`.
- **AWS Context**: EKS automates multi-master setup.
- **Interview Relevance**: Questions like “How do you set up a multi-master cluster?” Describe steps.

#### 2.9 Investigating Multi-Master Setup in EKS
- **Why**: EKS-specific knowledge is valuable for AWS-focused roles.
- **What**: EKS manages control plane across AZs, exposing a single endpoint.
- **How**: Create with `eksctl create cluster --region=us-west-2`. Inspect with `aws eks describe-cluster`.
- **AWS Context**: Core to EKS deployments.
- **Interview Relevance**: Expect “How does EKS handle HA?” Mention managed control plane.

#### 2.10 Exploring Multi-Master Setup in Monzo
- **Why**: Real-world examples like Monzo demonstrate practical HA implementations.
- **What**: Monzo uses multi-master Kubernetes for banking services, ensuring uptime.
- **How**: Study Monzo’s blog posts for architecture (e.g., etcd replication, load balancers).
- **AWS Context**: Apply similar principles in EKS.
- **Interview Relevance**: Questions like “Can you cite a real-world Kubernetes HA setup?” Reference Monzo/EKS.

#### 2.11 Creating a 3-Node Cluster with Kubeadm
- **Why**: Hands-on cluster setup skills are highly valued in interviews.
- **What**: A small production-like cluster with one master and two workers.
- **How**: Run `kubeadm init` on master, `kubeadm join` on workers. Install CNI (e.g., Calico).
- **AWS Context**: Simulate EKS setup locally before deploying.
- **Interview Relevance**: Be ready for “How do you set up a Kubernetes cluster?” Describe kubeadm steps.

#### 2.12 Installing an Overlay Network
- **Why**: Overlay networks enable pod-to-pod communication, a networking fundamental.
- **What**: CNI plugins like Calico or Flannel provide virtual networks.
- **How**: Apply with `kubectl apply -f calico.yaml`. Verify with `kubectl get pods -n kube-system`.
- **AWS Context**: EKS uses AWS VPC CNI by default.
- **Interview Relevance**: Questions like “What is a CNI plugin?” Explain overlay networks.

#### 2.13 Installing an Ingress Controller
- **Why**: Ingress controllers enable external access, critical for web apps.
- **What**: Controllers like NGINX or AWS ALB route HTTP traffic.
- **How**: Install with `helm install nginx-ingress ingress-nginx/ingress-nginx`. Configure Ingress resources.
- **AWS Context**: Use AWS ALB Ingress Controller in EKS.
- **Interview Relevance**: Expect “How do you set up Ingress?” Describe installation.

#### 2.14 Exploring the API Without kubectl
- **Why**: Direct API interaction demonstrates deep Kubernetes knowledge.
- **What**: Use `curl` or SDKs to query Kubernetes API server.
- **How**: Example: `curl -H "Authorization: Bearer <token>" https://api-server/pods`. Get token from `kubectl config`.
- **AWS Context**: Access EKS API with IAM credentials.
- **Interview Relevance**: Questions like “How do you interact with Kubernetes API?” Mention `curl`.

#### 2.15 Taking Down the Cluster One Node at a Time
- **Why**: Failure simulation tests resiliency, a key DevOps skill.
- **What**: Remove nodes to observe Kubernetes’ self-healing.
- **How**: Use `kubectl drain node` or `kubectl delete node`. Monitor Pod rescheduling.
- **AWS Context**: Test EKS resiliency with Cluster Autoscaler.
- **Interview Relevance**: Be ready for “How do you test cluster resiliency?” Describe simulation.

## Day 3: Networking and Advanced Scheduling

### 1. Cluster Networking and East-West Traffic

#### 1.1 Network Challenge with Multiple Nodes
- **Why**: Multi-node networking is complex, requiring robust solutions for pod communication.
- **What**: Pods on different nodes must communicate seamlessly.
- **How**: CNI plugins assign unique Pod IPs and route traffic. Example: Calico uses BGP.
- **AWS Context**: AWS VPC CNI integrates with VPC routing.
- **Interview Relevance**: Questions like “How do Pods communicate across nodes?” Explain CNI.

#### 1.2 Network Routing Recap
- **Why**: Understanding routing clarifies Kubernetes networking.
- **What**: Packets are routed via overlay networks or host networking.
- **How**: Inspect routing tables with `ip route` on nodes. CNI plugins manage routes.
- **AWS Context**: EKS uses VPC routing for efficiency.
- **Interview Relevance**: Expect “How does Kubernetes route traffic?” Describe CNI role.

#### 1.3 Understanding Network Requirements
- **Why**: Meeting network requirements ensures cluster functionality.
- **What**: Pods need unique IPs, Services need stable IPs, and nodes need connectivity.
- **How**: Configure CNI to allocate IP ranges (e.g., `10.244.0.0/16` for Flannel).
- **AWS Context**: EKS VPC CNI uses VPC CIDR.
- **Interview Relevance**: Questions like “What are Kubernetes network requirements?” List IPs.

#### 1.4 Understanding Cluster Address Limits
- **Why**: IP exhaustion can disrupt scaling, a common production issue.
- **What**: Limited by CNI and cluster size (e.g., Flannel’s /16 limit).
- **How**: Monitor with `kubectl get pods -o wide`. Adjust CNI config for larger ranges.
- **AWS Context**: EKS VPC CNI scales with VPC subnets.
- **Interview Relevance**: Be ready for “How do you handle IP exhaustion?” Explain CNI tuning.

#### 1.5 Assigning IP Addresses
- **Why**: IP assignment enables pod communication, a networking core.
- **What**: CNI plugins assign IPs from a pool to Pods/Services.
- **How**: Example: Calico assigns `192.168.x.x`. Check with `kubectl get pods -o wide`.
- **AWS Context**: EKS uses VPC IPs for Pods.
- **Interview Relevance**: Questions like “How are IPs assigned in Kubernetes?” Describe CNI.

#### 1.6 The Challenge of Service Discovery
- **Why**: Service discovery simplifies microservices communication.
- **What**: Services provide stable DNS names for Pods.
- **How**: CoreDNS resolves Service names (e.g., `myapp.default.svc.cluster.local`). Create with `kubectl expose`.
- **AWS Context**: EKS uses CoreDNS by default.
- **Interview Relevance**: Expect “How does service discovery work?” Mention DNS.

#### 1.7 Exploring the Endpoints
- **Why**: Endpoints map Services to Pods, critical for traffic routing.
- **What**: A resource listing Pod IPs for a Service.
- **How**: Check with `kubectl get endpoints`. Create Services to auto-populate.
- **AWS Context**: EKS Services manage endpoints dynamically.
- **Interview Relevance**: Questions like “What are Endpoints?” Explain mapping.

#### 1.8 Balancing In-Cluster Traffic
- **Why**: Load balancing ensures even traffic distribution, improving performance.
- **What**: Kube-proxy balances traffic to Service endpoints.
- **How**: Uses iptables or IPVS modes. Configure with `kube-proxy` settings.
- **AWS Context**: EKS kube-proxy integrates with VPC.
- **Interview Relevance**: Be ready for “How does Kubernetes balance traffic?” Describe kube-proxy.

#### 1.9 Routing Traffic with Kube-Proxy
- **Why**: Kube-proxy is central to Kubernetes networking, a key concept.
- **What**: Runs on each node, managing Service traffic routing.
- **How**: Inspect rules with `iptables -L`. Configure mode with `kube-proxy --proxy-mode=ipvs`.
- **AWS Context**: EKS optimizes kube-proxy for VPC.
- **Interview Relevance**: Questions like “What does kube-proxy do?” Explain routing.

#### 1.10 Discovering Service Types: Headless and ClusterIP
- **Why**: Service types control access patterns, essential for application design.
- **What**: ClusterIP is default (internal), Headless skips VIP for direct Pod access.
- **How**: Create with `kubectl expose --type=ClusterIP` or `clusterIP: None` for headless.
- **AWS Context**: Use ClusterIP in EKS for internal services.
- **Interview Relevance**: Expect “What are Service types?” List ClusterIP/Headless.

#### 1.11 Network Security for East-West Traffic
- **Why**: Securing internal traffic prevents unauthorized access, a security best practice.
- **What**: Network Policies restrict pod-to-pod communication.
- **How**: Apply with `kubectl apply -f networkpolicy.yaml` (e.g., allow specific labels).
- **AWS Context**: Use Calico in EKS for Network Policies.
- **Interview Relevance**: Questions like “How do you secure east-west traffic?” Mention Network Policies.

#### 1.12 Network Policy Targets
- **Why**: Precise targeting enhances security granularity.
- **What**: Policies use labels/namespaces to select Pods.
- **How**: Example: `podSelector: matchLabels: app=myapp` in NetworkPolicy YAML.
- **AWS Context**: Apply in EKS for microservices.
- **Interview Relevance**: Be ready for “How do Network Policies target Pods?” Explain selectors.

#### 1.13 Network Policy Best Practices
- **Why**: Best practices ensure effective and maintainable security.
- **What**: Use default-deny policies, limit scope, and test thoroughly.
- **How**: Define `policyTypes: Ingress, Egress` for explicit control. Test with `kubectl exec`.
- **AWS Context**: Implement in EKS with Calico.
- **Interview Relevance**: Questions like “What are Network Policy best practices?” List strategies.

### 2. North-South Traffic and Service Meshes

#### 2.1 Layer 4 Solution to Expose a Service
- **Why**: Layer 4 exposure enables TCP/UDP access, suitable for non-HTTP apps.
- **What**: NodePort or LoadBalancer Services expose Pods externally.
- **How**: Create with `kubectl expose --type=LoadBalancer`. NodePort uses ports 30000-32767.
- **AWS Context**: EKS provisions AWS ELB for LoadBalancer.
- **Interview Relevance**: Expect “How do you expose a TCP service?” Describe Service types.

#### 2.2 The Details of Using Nodeport
- **Why**: NodePort is simple for testing but limited for production, showing nuanced knowledge.
- **What**: Exposes a Service on each node’s IP at a high port.
- **How**: Create with `kubectl expose --type=NodePort`. Access via `<node-ip>:<node-port>`.
- **AWS Context**: Rarely used in EKS due to LoadBalancer preference.
- **Interview Relevance**: Questions like “What is NodePort?” Explain use case.

#### 2.3 Provision Loadbalancers Automatically
- **Why**: Load balancers simplify external access, a production standard.
- **What**: Kubernetes integrates with cloud providers to provision LBs.
- **How**: Use `type: LoadBalancer` in Service YAML. In EKS, this creates an ELB.
- **AWS Context**: EKS supports ALB/ELB for LoadBalancer.
- **Interview Relevance**: Be ready for “How do you provision a load balancer?” Mention Service.

#### 2.4 Helping Out the World to Save on IPs
- **Why**: IP conservation is critical in large clusters, showing scalability awareness.
- **What**: Ingress and Service meshes reduce public IP usage.
- **How**: Use Ingress to route multiple domains via one IP. Example: NGINX Ingress.
- **AWS Context**: EKS ALB Ingress conserves IPs.
- **Interview Relevance**: Questions like “How do you save IPs in Kubernetes?” Explain Ingress.

#### 2.5 HTTP Based Routing for Domains and Paths
- **Why**: HTTP routing enables flexible web app access, a common requirement.
- **What**: Ingress routes traffic based on domain/path.
- **How**: Define in `ingress.yaml` (e.g., `host: myapp.com, path: /api`). Apply with `kubectl apply`.
- **AWS Context**: Use AWS ALB Ingress in EKS.
- **Interview Relevance**: Expect “How does Ingress routing work?” Provide YAML.

#### 2.6 Routing Traffic with Ingress Manifests
- **Why**: Ingress manifests are declarative, aligning with Kubernetes philosophy.
- **What**: YAML files defining routing rules.
- **How**: Example: `rules: host: myapp.com, http.paths.path: /`. Apply with `kubectl apply`.
- **AWS Context**: EKS ALB Ingress uses annotations.
- **Interview Relevance**: Questions like “How do you configure Ingress?” Show YAML.

#### 2.7 The Battle of Ingress Controllers
- **Why**: Knowing controller options demonstrates practical experience.
- **What**: NGINX, Traefik, and AWS ALB are popular controllers.
- **How**: Install with Helm (e.g., `helm install nginx-ingress`). Compare features like SSL termination.
- **AWS Context**: EKS prefers AWS ALB Ingress.
- **Interview Relevance**: Be ready for “What Ingress controllers have you used?” Mention ALB.

#### 2.8 End-to-End Packet Journey
- **Why**: Understanding packet flow clarifies networking, a complex topic.
- **What**: Packets travel from client to Ingress, Service, and Pod.
- **How**: Trace with `tcpdump` on nodes or Istio telemetry. Example: Client → ALB → Pod.
- **AWS Context**: EKS integrates with VPC routing.
- **Interview Relevance**: Questions like “Describe a packet’s journey in Kubernetes?” Explain flow.

#### 2.9 Service Mesh Use-Cases
- **Why**: Service meshes enhance microservices, a trending DevOps topic.
- **What**: Provide mTLS, observability, and traffic control.
- **How**: Deploy Istio with `istioctl install`. Configure VirtualServices for routing.
- **AWS Context**: Use Istio in EKS for microservices.
- **Interview Relevance**: Expect “What are service mesh use cases?” List benefits.

#### 2.10 The Ins and Outs of Service Meshes
- **Why**: Deep service mesh knowledge sets you apart in interviews.
- **What**: Sidecars (e.g., Envoy) handle traffic, metrics, and security.
- **How**: Istio injects Envoy to Pods, enabling mTLS and tracing. Monitor with Kiali.
- **AWS Context**: Integrate Istio with EKS and CloudWatch.
- **Interview Relevance**: Questions like “How does a service mesh work?” Describe sidecars.

### 3. Advanced Scheduling

#### 3.1 How the Scheduler Works in Kubernetes
- **Why**: Scheduling is core to resource optimization, a DevOps responsibility.
- **What**: Filters nodes (resources, taints) and scores them for Pod placement.
- **How**: Inspect with `kubectl describe pod` (scheduling events). Tune with affinity rules.
- **AWS Context**: EKS scheduler optimizes EC2/Fargate.
- **Interview Relevance**: Be ready for “How does the scheduler work?” Explain filters.

#### 3.2 Filters and Predicates
- **Why**: Filters ensure valid node selection, critical for correct scheduling.
- **What**: Rules like resource availability or taint checks.
- **How**: Scheduler applies predicates (e.g., `PodFitsResources`). View logs for debugging.
- **AWS Context**: EKS applies filters to instance types.
- **Interview Relevance**: Questions like “What are scheduler predicates?” List examples.

#### 3.3 nodeSelector
- **Why**: nodeSelector provides simple node targeting, useful for basic scheduling.
- **What**: Matches Pods to nodes with specific labels.
- **How**: Define in Pod spec: `nodeSelector: disktype=ssd`. Apply with `kubectl apply`.
- **AWS Context**: Use in EKS for instance type targeting.
- **Interview Relevance**: Expect “What is nodeSelector?” Provide YAML.

#### 3.4 Node Affinity
- **Why**: Affinity offers flexible scheduling, improving workload placement.
- **What**: Rules to prefer or require nodes (e.g., `preferredDuringScheduling`).
- **How**: Define in YAML: `affinity.nodeAffinity.requiredDuringScheduling`. Apply with `kubectl apply`.
- **AWS Context**: Use in EKS for AZ placement.
- **Interview Relevance**: Questions like “How does node affinity work?” Compare to nodeSelector.

#### 3.5 Pod Affinity/Anti-Affinity
- **Why**: Pod affinity controls co-location, critical for performance or isolation.
- **What**: Rules to place Pods together or apart based on labels.
- **How**: Define in YAML: `affinity.podAffinity.requiredDuringScheduling`. Example: Co-locate cache Pods.
- **AWS Context**: Use in EKS for microservices.
- **Interview Relevance**: Be ready for “What is pod affinity?” Explain use cases.

#### 3.6 Taints and Tolerations
- **Why**: Taints restrict node access, enabling dedicated workloads.
- **What**: Taints repel Pods unless they have matching tolerations.
- **How**: Apply with `kubectl taint nodes node1 key=value:NoSchedule`. Add toleration in Pod spec.
- **AWS Context**: Use in EKS for GPU nodes.
- **Interview Relevance**: Questions like “How do taints work?” Describe process.

#### 3.7 Topology Spread Constraints
- **Why**: Spread constraints ensure HA and load distribution, a production best practice.
- **What**: Distribute Pods across zones or nodes.
- **How**: Define in YAML: `topologySpreadConstraints.maxSkew=1`. Apply with `kubectl apply`.
- **AWS Context**: Use in EKS for multi-AZ deployments.
- **Interview Relevance**: Expect “What are topology spread constraints?” Explain HA.

#### 3.8 Custom Schedulers
- **Why**: Custom schedulers address specialized needs, showing advanced knowledge.
- **What**: User-defined schedulers for specific workloads (e.g., ML).
- **How**: Write a scheduler using client-go, deploy as a Pod. Configure Pods to use it.
- **AWS Context**: Rarely used in EKS but possible for niche cases.
- **Interview Relevance**: Questions like “What is a custom scheduler?” Describe implementation.

## Day 4: Managing State, Autoscaling, and Security

### 1. Managing State in Kubernetes

#### 1.1 Managing Configurations
- **Why**: Configurations enable dynamic app settings, a DevOps necessity.
- **What**: ConfigMaps store key-value pairs or files.
- **How**: Create with `kubectl create configmap myconfig --from-literal=key=value`. Mount in Pods.
- **AWS Context**: Use in EKS for app settings.
- **Interview Relevance**: Be ready for “How do you manage configs?” Mention ConfigMaps.

#### 1.2 Managing Secrets
- **Why**: Secrets secure sensitive data, critical for compliance.
- **What**: Store encrypted data (e.g., passwords).
- **How**: Create with `kubectl create secret generic mysecret --from-literal=password=123`. Mount in Pods.
- **AWS Context**: Use AWS Secrets Manager with EKS.
- **Interview Relevance**: Questions like “How do you handle secrets?” Describe Secrets.

#### 1.3 Using Kubernetes Volumes
- **Why**: Volumes persist data, enabling stateful apps.
- **What**: Attach storage to Pods (e.g., `emptyDir`, `hostPath`).
- **How**: Define in Pod spec: `volumes: name=myvolume, emptyDir: {}`. Mount in containers.
- **AWS Context**: Use EBS/EFS in EKS.
- **Interview Relevance**: Expect “What are Kubernetes volumes?” List types.

#### 1.4 Creating Persistent Volumes
- **Why**: PVs provide durable storage, essential for databases.
- **What**: Define storage resources (e.g., NFS, EBS).
- **How**: Create with `kubectl apply -f pv.yaml` (e.g., `storage: 10Gi, accessModes: ReadWriteOnce`).
- **AWS Context**: Use EBS in EKS.
- **Interview Relevance**: Questions like “What is a PV?” Provide YAML.

#### 1.5 Creating Persistent Volume Claims
- **Why**: PVCs abstract storage requests, simplifying Pod configuration.
- **What**: Requests for PVs by Pods.
- **How**: Create with `kubectl apply -f pvc.yaml` (e.g., `storage: 10Gi`). Bind to PVs.
- **AWS Context**: Use PVCs with EBS in EKS.
- **Interview Relevance**: Be ready for “What is a PVC?” Explain binding.

#### 1.6 Provisioning Volumes Dynamically
- **Why**: Dynamic provisioning automates storage, improving scalability.
- **What**: StorageClasses trigger volume creation.
- **How**: Define in `storageclass.yaml` (e.g., `provisioner: kubernetes.io/aws-ebs`). Use in PVCs.
- **AWS Context**: EKS uses EBS CSI driver.
- **Interview Relevance**: Questions like “How do you provision storage dynamically?” Mention StorageClasses.

#### 1.7 Managing Stateful Applications
- **Why**: Stateful apps require stable identities, a complex Kubernetes use case.
- **What**: Use StatefulSets for apps like databases.
- **How**: Create with `kubectl apply -f statefulset.yaml` (e.g., `serviceName: mysql`).
- **AWS Context**: Deploy MySQL in EKS with EBS.
- **Interview Relevance**: Expect “How do you manage stateful apps?” Describe StatefulSets.

#### 1.8 Deploying a Single Database with Persistence
- **Why**: Databases are common stateful workloads, testing practical skills.
- **What**: Run a single-instance DB with persistent storage.
- **How**: Use StatefulSet with PVC. Example: MySQL with EBS volume.
- **AWS Context**: Deploy in EKS with EBS CSI.
- **Interview Relevance**: Questions like “How do you deploy a database?” Provide YAML.

#### 1.9 Deploying a Clustered Database with Persistence
- **Why**: Clustered DBs test advanced state management, a senior-level skill.
- **What**: Run replicated DBs with stable identities.
- **How**: Use StatefulSet with headless Service and PVCs. Configure DB replication (e.g., MySQL Galera).
- **AWS Context**: Use EFS for shared storage in EKS.
- **Interview Relevance**: Be ready for “How do you deploy a clustered DB?” Explain setup.

### 2. Autoscaling

#### 2.1 Introduction to Autoscaling
- **Why**: Autoscaling ensures performance and cost-efficiency, a production must-have.
- **What**: Adjusts Pod or node counts based on demand.
- **How**: Use HPA for Pods, Cluster Autoscaler for nodes. Configure with `kubectl autoscale`.
- **AWS Context**: EKS supports HPA and Cluster Autoscaler.
- **Interview Relevance**: Questions like “What is autoscaling in Kubernetes?” List types.

#### 2.2 The Horizontal Pod Autoscaler
- **Why**: HPA automates Pod scaling, reducing manual effort.
- **What**: Scales Pods based on CPU/memory or custom metrics.
- **How**: Create with `kubectl autoscale deployment myapp --min=2 --max=10 --cpu-percent=80`.
- **AWS Context**: Use with EKS Metrics Server.
- **Interview Relevance**: Expect “How does HPA work?” Describe metrics.

#### 2.3 The Kubernetes Metrics Registry
- **Why**: Metrics drive autoscaling, critical for observability.
- **What**: Metrics Server collects resource usage (CPU/memory).
- **How**: Install with `kubectl apply -f metrics-server.yaml`. Query with `kubectl top`.
- **AWS Context**: EKS includes Metrics Server.
- **Interview Relevance**: Questions like “What is the Metrics Server?” Explain role.

#### 2.4 Exposing Metrics from Your Apps
- **Why**: Custom metrics enable app-specific scaling, a senior skill.
- **What**: Apps expose metrics via endpoints (e.g., `/metrics`).
- **How**: Use Prometheus client libraries (e.g., prom-client for Node.js). Scrape with Prometheus.
- **AWS Context**: Integrate with EKS Prometheus.
- **Interview Relevance**: Be ready for “How do you expose custom metrics?” Describe process.

#### 2.5 Installing and Configuring Prometheus
- **Why**: Prometheus is a standard for Kubernetes monitoring, widely used in DevOps.
- **What**: Collects and stores time-series metrics.
- **How**: Install with `helm install prometheus prometheus-community/prometheus`. Configure scrape targets.
- **AWS Context**: Use with EKS and CloudWatch.
- **Interview Relevance**: Questions like “How do you set up Prometheus?” Mention Helm.

#### 2.6 Understanding Custom and External Metrics Adapters
- **Why**: Adapters enable advanced autoscaling, showcasing expertise.
- **What**: Bridge Prometheus or external sources to HPA.
- **How**: Deploy Prometheus Adapter with `helm install prometheus-adapter`. Configure HPA with `metrics: type=Pods`.
- **AWS Context**: Use in EKS for custom scaling.
- **Interview Relevance**: Expect “What are custom metrics adapters?” Explain role.

#### 2.7 Tuning the Horizontal Pod Autoscaler
- **Why**: Tuning optimizes scaling behavior, critical for production.
- **What**: Adjust parameters like stabilization window or metric thresholds.
- **How**: Configure in HPA YAML: `behavior.scaleDown.stabilizationWindowSeconds=300`.
- **AWS Context**: Tune in EKS for cost/performance balance.
- **Interview Relevance**: Questions like “How do you tune HPA?” Describe parameters.

### 3. Security

#### 3.1 Attack Matrix of Threats
- **Why**: Understanding threats informs security strategies, a senior responsibility.
- **What**: Risks like container escapes, unauthorized access, or misconfigurations.
- **How**: Use MITRE ATT&CK for Kubernetes to identify threats. Mitigate with RBAC, Network Policies.
- **AWS Context**: Apply in EKS with IAM and security groups.
- **Interview Relevance**: Be ready for “What are Kubernetes security threats?” List examples.

#### 3.2 Security for Docker Apps
- **Why**: Container security prevents vulnerabilities, critical for production.
- **What**: Run as non-root, use minimal images, scan for CVEs.
- **How**: Define in Dockerfile: `USER nobody`. Scan with Trivy. Use `docker run --user=1000`.
- **AWS Context**: Scan images in ECR with Inspector.
- **Interview Relevance**: Questions like “How do you secure Docker containers?” Mention best practices.

#### 3.3 Security for Kubernetes Clusters
- **Why**: Cluster security protects infrastructure, a DevOps priority.
- **What**: Enable RBAC, secure etcd, use namespaces, apply Pod Security Standards.
- **How**: Configure with `kubectl create role`. Secure API server with TLS.
- **AWS Context**: EKS integrates with IAM for RBAC.
- **Interview Relevance**: Expect “How do you secure a Kubernetes cluster?” List measures.

#### 3.4 Security for Kubernetes Networking
- **Why**: Network security prevents lateral movement, essential for microservices.
- **What**: Use Network Policies, mTLS, and encryption.
- **How**: Apply NetworkPolicy YAML. Deploy Istio for mTLS.
- **AWS Context**: Use Calico and AWS security groups in EKS.
- **Interview Relevance**: Questions like “How do you secure Kubernetes networking?” Describe policies.

#### 3.5 Authentication in Kubernetes
- **Why**: Authentication controls access, foundational for security.
- **What**: Supports OIDC, certificates, or tokens.
- **How**: Configure API server with `--oidc-issuer-url`. Use `kubectl config` for client certs.
- **AWS Context**: EKS uses IAM for authentication.
- **Interview Relevance**: Be ready for “How does Kubernetes authentication work?” Mention methods.

#### 3.6 Role-Based Access Control
- **Why**: RBAC ensures least privilege, a security best practice.
- **What**: Roles/ClusterRoles define permissions, bound to users via RoleBindings.
- **How**: Create with `kubectl create role pod-reader --verb=get --resource=pods`. Bind with `kubectl create rolebinding`.
- **AWS Context**: Map IAM roles to EKS RBAC.
- **Interview Relevance**: Questions like “How does RBAC work?” Provide YAML.

## Additional Topics

### Service Meshes (and Istio)
- **Why**: Istio enhances microservices, a trending skill for senior DevOps roles.
- **What**: Provides mTLS, traffic routing, observability via Envoy sidecars.
- **How**: Install with `istioctl install`. Configure VirtualServices for routing. Monitor with Kiali/Prometheus.
- **AWS Context**: Deploy Istio in EKS for microservices.
- **Interview Relevance**: Expect “What is Istio?” Describe features and setup.

### CKA/CKAD Exam Preparation
- **Why**: Certifications validate hands-on Kubernetes skills, boosting interview credibility.
- **What**: CKA focuses on admin tasks (e.g., cluster setup), CKAD on app deployment.
- **How**: Practice with `kubectl`, debug Pods, create RBAC. Use tools like kubeadm, Minikube.
- **AWS Context**: Apply skills to EKS management.
- **Interview Relevance**: Questions like “How did you prepare for CKA/CKAD?” List tasks.

### Logging and Monitoring
- **Why**: Observability ensures reliability, a core DevOps responsibility.
- **What**: Use EFK for logging, Prometheus/Grafana for monitoring.
- **How**: Deploy Fluentd with `helm install fluentd`. Install Prometheus/Grafana. Configure log aggregation.
- **AWS Context**: Use CloudWatch for EKS logging/monitoring.
- **Interview Relevance**: Be ready for “How do you monitor Kubernetes?” Mention tools.

### Advanced Scheduling Workloads
- **Why**: Advanced scheduling optimizes specialized workloads, a senior skill.
- **What**: Use affinity, taints, or custom schedulers for ML/GPU workloads.
- **How**: Define in YAML: `nodeAffinity.requiredDuringScheduling`. Deploy NVIDIA plugins for GPUs.
- **AWS Context**: Schedule on EKS GPU instances (e.g., g4dn).
- **Interview Relevance**: Questions like “How do you schedule ML workloads?” Describe affinity/taints.
