# Backend Track Report

## Part A

Selected track: backend.

Component being deployed: the SpeedScore backend REST API. This is the Express and MongoDB API service that provides user, round, and authentication routes for SpeedScore.

Deployment priorities:

- Low cost: the deployment should be completed without paid infrastructure because this is a short class lab and does not require production-scale capacity.
- Operational simplicity: the deployment should be understandable, repeatable, and avoid unnecessary server administration.
- Reliability and verification: the deployed API should expose a clear health check so the deployment can be verified through a public endpoint and a deployment workflow.

I selected the backend track because the SpeedScore API is the component responsible for server-side application behavior, authentication, persistence, and runtime configuration. Deploying the backend gives me practice with the infrastructure and environment-variable concerns that are central to making a web application usable outside a local development machine.

## Part B

Deployment options compared:

1. Managed platform deployment, such as a Render or Railway web service.
2. IaaS container deployment, such as Docker Compose on an AWS EC2, Azure VM, Google Compute Engine, or DigitalOcean droplet.
3. Orchestrated container deployment, such as Kubernetes on AWS EKS, Google Kubernetes Engine, or Azure Kubernetes Service managed with Pulumi and GitHub Actions.

How each option could be implemented:

| Option | Tools/platforms needed | Basic implementation path |
| --- | --- | --- |
| Managed platform deployment | Render or Railway, GitHub, Node/Express start script, environment variables, health check path | Connect the GitHub repository to the platform, configure the backend as a web service, add required environment variables such as `MONGODB_URI`, `SESSION_SECRET`, `JWT_SECRET`, `NODE_ENV`, and `API_DEPLOYMENT_URL`, set the health check route to `/health`, and let the platform build and run the service. GitHub Actions can trigger a deploy hook and verify the public health endpoint. |
| IaaS container deployment | AWS EC2, Azure VM, Google Compute Engine, or DigitalOcean droplet; Docker; Docker Compose; SSH keys; GitHub Actions secrets; optional Nginx/Caddy for TLS | Provision a VM, install Docker and Docker Compose, copy or pull the backend Compose configuration onto the server, configure environment variables on the host, and use GitHub Actions to SSH into the VM after each push. The workflow would pull the newest image, restart the service with Docker Compose, and verify the health endpoint. |
| Orchestrated container deployment | Kubernetes through EKS, GKE, or AKS; Docker; container registry; Pulumi; GitHub Actions; Kubernetes manifests or Pulumi Kubernetes resources | Build the backend container in GitHub Actions, push it to a registry, and use Pulumi to create or update the Kubernetes cluster and application resources. Kubernetes would manage pods, service routing, health checks, restarts, scaling, secrets, and rolling updates. |

Tradeoff matrix:

| Dimension | Managed platform deployment | IaaS container deployment | Orchestrated container deployment |
| --- | --- | --- | --- |
| Infrastructure responsibility | Low. This matches the PaaS idea from the exploration because the platform manages most hosting, routing, TLS, scaling, and runtime operations. | High. This matches the IaaS model because the team rents a VM but still manages the operating system, Docker runtime, firewall rules, TLS, monitoring, and recovery. | Medium to high. The cloud provider can manage the control plane, but the team still manages cluster configuration, Kubernetes resources, images, secrets, networking, and deployment policy. |
| Deployment-path automation | High. GitHub Actions can trigger the platform's deploy hook and verify the health endpoint with relatively few steps. | Medium. GitHub Actions can SSH into the server and run Docker Compose commands, but this creates more manual credential and server-state risk. | High once configured. GitHub Actions and Pulumi can build images, push to a registry, update infrastructure, and apply Kubernetes deployment changes. |
| Operational complexity | Low to medium. Most routine operations are handled by the platform, though the team must still configure environment variables and health checks correctly. | High. The single-server model is simple conceptually, but ongoing maintenance is heavier because the server itself becomes part of the application. | High. Kubernetes adds pods, services, deployments, cluster permissions, and orchestration behavior that are powerful but complex for a small backend. |
| Reliability and health verification | Good for a class project because managed platforms usually support health checks, logs, automatic restarts, and stable generated URLs. | Depends heavily on manual setup. Docker restart policies help, but reliability is limited by single-server failure and manual maintenance. | Strong when configured correctly because Kubernetes supports health checks, restarts, scaling, service routing, and rolling updates. |
| Portability and coupling | Medium. The app remains portable as a Node/Express backend, but deploy behavior can become tied to platform-specific settings. | Medium to high. Docker Compose is portable across hosts, but server setup steps can become manual and environment-specific. | Medium. Kubernetes manifests and container images are portable across clusters, but the implementation is still coupled to Kubernetes concepts and provider-specific cluster setup. |
| Cost fit for lab scope | Strong. A Render Free web service can satisfy the lab evidence requirement without paid infrastructure if the app stays within free-tier limits. | Mixed. A small VM may be free or cheap on some providers, but billing-account setup and network limits make it riskier for a no-cost class deployment. | Weak. Managed Kubernetes clusters often introduce cost, quota, and setup overhead that are hard to justify for a small lab backend. |
| Control | Medium. The platform hides many infrastructure details, which reduces setup work but limits customization. | High. The team controls the VM, Docker runtime, ports, reverse proxy, and update process directly. | Very high. The team controls deployment strategy, scaling, health checks, networking, and rollout behavior, but this control comes with significant configuration overhead. |

The most important dimensions for this lab are low cost, operational simplicity, reliability/verification, and maintainability. Cost matters most because the deployment only needs to prove that the backend can run and be verified; it does not need production-grade capacity or a custom domain.

Recommendation: use a managed platform deployment, specifically a Render Free web service if the backend can be deployed within Render's free-tier limits. This is the most practical completely free option for this lab because Render can host a small backend web service without requiring a paid VM or Kubernetes cluster, and it provides a public URL that can be used for health-check evidence. The IaaS and Kubernetes options connect strongly to the exploration topics on virtual machines, Docker Compose, infrastructure as code, and orchestration, but they add more setup and cost risk than this lab requires. The main tradeoff is that Render Free services can spin down after inactivity, so the first health-check request after an idle period may be slower, but that limitation is acceptable for a class lab where the goal is deployment evidence rather than production uptime.

## Part C

## Part D

## Part E
