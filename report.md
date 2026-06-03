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

Selected deployment option: managed platform deployment using Render Free web service.

Deployment pipeline evidence:

- Deployment repository used by Render: https://github.com/RayHao2/speedscore-api-render-lab9
- GitHub Actions workflow run: https://github.com/RayHao2/speedscore-api-render-lab9/actions/runs/26859893291

Verification evidence:

- Public API health check URL: https://lab9-vc5q.onrender.com/health
- Health check output:

```json
{
  "status": "ok",
  "service": "speedscore-backend"
}
```

Scaffold changes made:

- Added `mongodb@6.3.0` as an explicit production dependency because `src/server.js` directly imports `MongoClient` from `mongodb`. Pinning version `6.3.0` keeps it compatible with `connect-mongo`.
- Kept the case-sensitive middleware filename aligned as `src/middleware/rateLimiter.js` because Linux hosts such as Render are case-sensitive and the server imports `./middleware/rateLimiter.js`.
- Used the existing `/health` endpoint in `src/server.js` as the deployment verification route.
- Added a GitHub Actions deployment workflow that installs dependencies, triggers the Render deploy hook, and verifies the deployed `/health` endpoint.
- Configured a MongoDB Atlas free cluster and allowed network access from Render so the deployed backend can connect to MongoDB.
- Configured Render environment variables for the database connection, session secret, JWT secret, token durations, production mode, deployment URL, and placeholder GitHub OAuth credentials.

Deployment note:

Render was configured as a Node web service from the repository root with `npm install` as the build command, `npm start` as the start command, and `/health` as the health check path. The service runs on Render Free, so it may spin down after inactivity, but it satisfies the lab requirement for a deployed backend with public verification evidence.

## Part D

Major steps attempted:

1. I reviewed the updated backend starter repository and confirmed that it now contains the real SpeedScore Express/MongoDB backend instead of only deployment scaffolding.
2. I selected Render Free as the managed platform deployment target and MongoDB Atlas free tier as the database service.
3. I added `mongodb@6.3.0` as an explicit dependency after noticing that `src/server.js` directly imports `MongoClient`.
4. I created a MongoDB Atlas free cluster, database user, and network access rule so the deployed backend could connect to MongoDB.
5. I created a local `.env` file and tested the backend locally with `npm start`.
6. I configured a Render Free Node web service from the repository root using `npm install`, `npm start`, and `/health`.
7. I added a GitHub Actions workflow that triggers the Render deploy hook and verifies the deployed `/health` endpoint.
8. I collected the successful workflow run link and the public Render health check output as deployment evidence.

Pain points, gotchas, and failures:

- Installing `mongodb` without a version first installed `mongodb@7.2.0`, which conflicted with `connect-mongo` because `connect-mongo@5.1.0` requires a MongoDB driver version below 7.
- The backend could not start at first because the GitHub OAuth Passport strategy requires `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET` at startup, even though GitHub OAuth was not needed for the health-check deployment.
- The updated backend requires a real MongoDB connection string because the server connects to MongoDB and creates a session store during startup.
- Render initially showed Docker settings, but the updated repository does not include a Dockerfile. The correct setup was a Node web service from the repository root.
- Render initially selected Node 24, which caused a MongoDB TLS connection failure during deployment. Pinning the runtime to Node 20 fixed the production startup issue.
- The Render Free service may spin down after inactivity, which can make the first verification request slower.

Responses and lessons learned:

- To fix the MongoDB dependency conflict, I pinned the explicit dependency to `mongodb@6.3.0`, which matches the version already used by Mongoose and stays compatible with `connect-mongo`.
- To get the server running without implementing real GitHub OAuth for this lab, I provided placeholder `GITHUB_CLIENT_ID` and `GITHUB_CLIENT_SECRET` values in local and Render environment variables.
- To support the real backend, I used MongoDB Atlas instead of a placeholder health-only API. This made the deployment match the updated starter repo more closely.
- To avoid unnecessary deployment complexity, I used Render's Node runtime rather than adding a Dockerfile.
- To fix the Render runtime issue, I pinned the Node version in `package.json` and configured Render to use Node 20 instead of Node 24.
- I learned that backend deployment depends heavily on environment configuration. The code can be correct locally, but missing secrets, dependency versions, database network access, or platform runtime settings can still prevent deployment.

Advice to my future self:

Before deploying this backend again, I would first list every environment variable used during startup, confirm dependency compatibility, and verify database access before creating the hosting service. I would also check whether the platform is using the correct runtime, because choosing Docker when the repo is set up for Node can send the deployment in the wrong direction.

## Part E

Backend deployment guide for future students:

Prerequisites:

- A GitHub repository containing the SpeedScore backend code.
- A Render account for the free web service.
- A MongoDB Atlas account with access to create a free cluster.
- Node.js installed locally.
- A `/health` endpoint in the backend that returns HTTP `200`.
- GitHub Actions secrets access for the repository used by Render.

Key setup and deployment steps:

1. Confirm the backend starts from the repository root with:

```powershell
npm install
npm start
```

2. Make sure `mongodb` is listed as an explicit dependency and use a version compatible with `connect-mongo`. For this repo, `mongodb@6.3.0` worked.

3. Create a MongoDB Atlas free cluster. Add a database user and allow network access from anywhere with `0.0.0.0/0`, since Render Free does not provide a stable outbound IP for simple allowlisting.

4. Create a local `.env` file for testing:

```env
MONGODB_URI=<your Atlas connection string>
SESSION_SECRET=<long random string>
JWT_SECRET=<long random string>
ACCESS_TOKEN_DURATION=1h
REFRESH_TOKEN_DURATION=7d
PORT=3001
NODE_ENV=development
API_DEPLOYMENT_URL=http://localhost:3001
GITHUB_CLIENT_ID=lab9-placeholder-client-id
GITHUB_CLIENT_SECRET=lab9-placeholder-client-secret
```

5. Test locally:

```powershell
npm start
curl.exe http://localhost:3001/health
```

6. Create a Render Web Service using the repository root:

```text
Runtime: Node
Root Directory: leave blank
Build Command: npm install
Start Command: npm start
Instance Type: Free
Health Check Path: /health
Auto-Deploy: Off
```

7. Add Render environment variables:

```text
MONGODB_URI
SESSION_SECRET
JWT_SECRET
ACCESS_TOKEN_DURATION=1h
REFRESH_TOKEN_DURATION=7d
NODE_ENV=production
API_DEPLOYMENT_URL=https://<your-render-service>.onrender.com
GITHUB_CLIENT_ID=lab9-placeholder-client-id
GITHUB_CLIENT_SECRET=lab9-placeholder-client-secret
```

8. Deploy the Render service and verify:

```powershell
curl.exe https://<your-render-service>.onrender.com/health
```

9. Add GitHub Actions secrets:

```text
RENDER_DEPLOY_HOOK_URL
RENDER_SERVICE_URL
```

10. Run the GitHub Actions deployment workflow and save the successful workflow run link.

Main gotchas:

- Do not commit `.env`; use Render environment variables and GitHub Actions secrets.
- Do not set `PORT` on Render; Render provides it automatically.
- Pin Node to version 20 for this backend; newer Render defaults such as Node 24 can introduce runtime or TLS compatibility problems.
- The service must use the Node runtime unless a Dockerfile has been intentionally added.
- The GitHub OAuth variables are needed at startup because Passport registers the GitHub strategy immediately.
- The MongoDB Atlas connection string should include a database name before the query string.
- Render Free services can spin down after inactivity, so a slow first request does not necessarily mean deployment failed.

Success verification checklist:

- Local `/health` returns HTTP `200`.
- Render logs show the server started successfully.
- Render `/health` returns:

```json
{
  "status": "ok",
  "service": "speedscore-backend"
}
```

- GitHub Actions workflow completes successfully.
- The lab report includes the workflow run link, public health check URL, health check output, and scaffold-change notes.
