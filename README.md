# catalogue

# RoboShop

### Create RoboShop Folder

* Create a **RoboShop** folder.
* Save it.

### Create Pipeline

* Go to **RoboShop** → **Create Pipeline**.

### Version Changes

Here, the versions are changing, so we need to read the version from the `package.json` file.

For this, we need to install a plugin in Jenkins:

* **Pipeline Utility Steps** plugin

### Dependency Installation

We need to log in to the agent and run the following commands:

```bash
dnf module disable nodejs
sudo dnf module enable nodejs:20 -y
dnf install nodejs -y
```
### Create Webhook

* Create a webhook → **Configure** → **Add** → **GitHub Hook**.

### Create Docker Image

Now we need to create a Docker image:

```bash
docker build -t URL/catalogue:appVersion
```

### Push Docker Image to ECR

Now we need to push the Docker image to **ECR**.

**AWS → Amazon ECR → Private Registry → Repository → Create**

* Repository name: `roboshop/catalogue`
* Image tag mutability: **Mutable**
* Click **Create**.

There is an **Instructions** tab that will show all the required steps.

### Create Credentials

1. We need to log in to AWS. For this, we need to create credentials.
2. Install the required plugin:

**Plugin → AWS Credentials**


## Jenkins Credentials

**Manage Jenkins → Credentials → Kind → AWS Credentials**

* ID: `aws-auth`
* SSH Credentials: `ssh-cre`
* Enter the **Access Key** and **Secret Key**.

### Plugin

Install the **AWS Steps** plugin.

This is used for **API calls**.

## Install Docker Plugin on Jenkins Agent

Run the following commands on the Jenkins agent:

```bash
sudo dnf -y install dnf-plugins-core

sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo

sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo systemctl start docker

sudo systemctl enable docker

sudo usermod -aG docker ec2-user
```

After adding the `ec2-user` to the Docker group:

* Disconnect from the agent.
* Connect to the agent again.

## Run Pipeline

* Run the pipeline.
* We can see the Docker image in **ECR**.
* Scan the image.

## Testing

### Functional Testing

**DEV → Developers / Testers**

Functional testing is performed by developers and testers.

### Integration Testing

**UAT**

Here, all components need to communicate properly.

## Deployment

Now the image is ready, so we need to deploy it.


## Create INFRA Folder

### Install Terraform on the Node

#### Install Required Utilities

```bash
sudo dnf install -y dnf-utils yum-utils
```

#### Add HashiCorp Repository

```bash
sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
```

#### Install Terraform

```bash
sudo dnf install -y terraform
```

#### Verify Installation

```bash
terraform -version
```

### Create Node

**`00-vpc` → Pipeline**

Configure:

**`00-vpc/jenkins`**

### Plugin

Install the **AnsiColor** plugin.

> If any pipeline fails after installing plugins, we need to restart Jenkins.

```bash
sudo systemctl restart jenkins
```



# UPSTREAM AND DOWNSTREAM

## Create Pipeline for Security Group

* Create a pipeline for the security group.
* Pipeline: **`10-sg`**
* Copy the downstream configuration from **`00-vpc`**.
* Follow the same process.

## Catalogue

* `catalogue` → For **CI**
* `catalogue-cd` → For **CD**

## Install kubectl on the Node

On the node side, we can install `kubectl`.

### Download kubectl

```bash
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.10/2026-04-08/bin/linux/amd64/kubectl
```

### Give Execute Permission

```bash
chmod +x kubectl
```

### Move kubectl

```bash
sudo mv kubectl /usr/local/bin/
```

### Verify Installation

```bash
kubectl version --client
```


## Create Pipelines for Catalogue

Create pipelines for **`catalogue-ci`** and **`catalogue-cd`** under the **RoboShop** folder.

### VPC Peering

Here, **VPC peering is required**.

### Plugin

Install the **Rebuilder** plugin for rebuilding the pipeline.

## EKS API Connection Error

We may get the following error:

```text
err="couldn't get current server API group list: Get \"https://104097364FA6AA759BD576D04FDCF3C2.gr7.us-east-1.eks.amazonaws.com/api?timeout=32s\": dial tcp 10.0.11.18:443: i/o timeout"
```

For this, we need to do something manually.

### Update Security Group

**EC2 → EKS → Networking → Cluster SG**

It will take us to the **VPC**.

**VPC → Security Groups → `roboshop-dev-controlplane` → Edit Inbound Rules**

1. Click **Add Rule**.
2. Select **HTTPS**.
3. Copy the **Agent Security Group ID**.
4. Add the **Agent SG ID** as the source.
5. Save the rule.

After this, we should get a successful connection.

## Catalogue

For **Catalogue**, MongoDB needs to be running.

## Deployment Process

1. Create the infrastructure.
2. Set up EKS.
3. Deploy the application.
4. **First deployment fails:** Helm cannot roll back because there will be no previous version (**revision 0**).
5. **Second deployment fails:** Helm attempts a rollback, but the rollback also fails.

   * Deployment failed.
   * Rollback failed.
6. **Third deployment succeeds.**

   * Third deployment: **Success**
7. **Fourth revision fails:** The deployment fails, but the Helm rollback is successful.

   * Deployment: **Failed**
   * Rollback: **Successful**

## Deployment and Rollback Flow

### 1st Deployment

```text
1st Deployment
     |
   FAILED
     |
No previous revision
     |
Rollback NOT possible
     |
Deployment Failed
```

### 2nd Deployment

```text
2nd Deployment
     |
   FAILED
     |
Rollback attempted
     |
Rollback FAILED
     |
Deployment Failed
Rollback Failed
```

### 3rd Deployment

```text
3rd Deployment
     |
   SUCCESS
     |
Deployment Successful
```

### 4th Deployment

```text
4th Deployment
     |
   FAILED
     |
Rollback attempted
     |
Rollback SUCCESS
     |
Deployment Failed
Rollback Successful
```


# Scans

## Shift Left

**Shift Left** means bringing testing and scanning to the early stages, such as the **DEV** stage, instead of doing them in higher environments.

## Build Once, Run Anywhere

We only build the application in the **DEV environment**. We don't build it in multiple environments.

We promote the same application to multiple environments with different configurations.

## Types of Scans

### Static Source Code Analysis

* **SonarQube**

### Static Application Security Testing (SAST)

* **SonarQube**
* **Fortify Scan**
* **GitHub**

### Open Source Library Scan

* **Nexus IQ**
* **GitHub Dependabot**

### Dynamic Application Security Testing (DAST)

Attacks are performed on a **running application**.

* **Fortify WebInspect**
* **Veracode**

## Docker Image Scan

A Docker image scan is used to identify **vulnerabilities in Docker images**.

## SonarQube

The installation and setup of **SonarQube** can be difficult, so we will use **AWS Marketplace SonarQube**.

**AWS → EC2 → SonarQube CE on AWS → Create Instance**

For **SonarQube**, the default port number is **9000**.


## Jenkins Server and Scanner

**Jenkins Server → Scanner Agent → Scanner Server**

We have the source code, and someone needs to analyze it. The **Scanner Agent** will scan the source code and send the results to the **Scanner Server**.

```text
Jenkins Server
      |
      | Source Code
      ↓
Scanner Agent
      |
      | Scan Results
      ↓
Scanner Server
      |
      | Analysis
      ↓
Quality Gate
      |
      ↓
Jenkins
```

### Process

* The **Scanner Agent** accesses the source code and performs the scan.
* The scan results are sent to the **central Scanner Server**.
* The actual analysis happens on the **Scanner Server**.
* After the analysis is completed, the Scanner Server sends the result back to **Jenkins**.
* Jenkins checks the result based on the configured **Quality Gate**.

### Important Point

The **Scanner Agent scans the code**, but the **actual analysis happens on the Scanner Server**.


## Jenkins and Scanner Flow

### What Is Correct in the Diagram?

* Jenkins triggers the process. ✔️
* The Scanner Agent runs inside Jenkins. ✔️
* The Scanner Agent scans the code. ✔️
* Results are sent to the Scanner Server. ✔️
* The Scanner Server performs the actual analysis. ✔️
* Quality Gates are involved. ✔️

### 🔧 Small Improvement

Right now, the diagram shows:

```text
Scanner Agent → Results → Scanner Server → Analysis
```

But one important step is missing visually.

After the server analysis:

**Scanner Server → Sends the result back to Jenkins → Jenkins checks the Quality Gate status**

### Complete Flow

```text
Jenkins
   ↓
Scanner Agent (scans code)
   ↓
Send results → Scanner Server
   ↓
Server performs full analysis
   ↓
Apply Quality Gates
   ↓
Send status back to Jenkins
   ↓
Jenkins decides (Pass ✅ / Fail ❌)
```

## Complete CI/CD Scanner Flow

```text
[Developer Pushes Code]
          ↓
    [Source Code Repo]
          ↓
      [Jenkins Server]
          ↓
   (Triggers Build Pipeline)
          ↓
   [Scanner Agent Runs]
   (e.g., Sonar Scanner)
          ↓
   → Scans the source code
   → Performs initial analysis
          ↓
  [Results Sent to Scanner Server]
  (Central Server - e.g., SonarQube)
          ↓
  [Server-Side Analysis Happens]
   → Deep analysis
   → Apply rules & standards
   → Compute metrics
     (bugs, vulnerabilities, code smells)
          ↓
   [Quality Gates Evaluation]
   → Pass / Fail based on conditions
          ↓
   [Response Sent Back to Jenkins]
          ↓
   [Jenkins Pipeline Decision]
   → If PASS → Continue build/deploy 🚀
   → If FAIL → Stop pipeline ❌
```

### Key Point

The **Scanner Agent performs the scanning and sends the results to the Scanner Server**.

The **Scanner Server performs the main analysis, evaluates the Quality Gate, and sends the status back to Jenkins**.

## SonarQube Configuration in Jenkins

### Install SonarQube Scanner Plugin

We need to install the **SonarQube Scanner** plugin.

It will enable the **SonarQube options** in Jenkins.

### Configure SonarQube Scanner

Go to:

**Manage Jenkins → Tools**

* Name: `sonar-7.0`
* Select **Install automatically**.

### Configure SonarQube Server

The agent needs to know where the **SonarQube server** is present.

We need to provide the SonarQube server URL in Jenkins.

Go to:

**Manage Jenkins → System → SonarQube Servers → Add SonarQube**

* Name: `sonar-7.2`
* Server URL: `http://<sonar_ip_pub>:9000`
* Add the token created in SonarQube.

In **SonarQube**, create a token and add it to Jenkins.

### Add SonarQube Stage

Now we will add a **SonarQube stage** to the **CI pipeline**.


## Quality Gates Strategy
static code analyis
### Code Scope Definition

* **Commits:** We have two commits: **C1** and **C2**
* **New Code:** `C2 - C1`
* **Overall Code:** `C1 + C2`

### Code Coverage Approach

We implemented code coverage analysis for:

* **Overall Code**
* **New Code**

### Implementation Steps

1. Enabled **SonarScanner** and **SonarQube Server** integration in Jenkins.
2. Requested all developers to onboard their projects into SonarQube.
3. Provided a **4-month timeline** to address all quality issues, including:

   * Code issues
   * Bugs
   * Vulnerabilities
   * Code smells
   * Maintainability rating
   * Security rating
     (for both overall code and new code)
4. we enaled qualty gate instially not aborting the pipeline
5. after 2 weeks we started aborting pipelines if quality gate
we need to create q webhook in sonarcube for jenkins 
beacuse jenkins by deafult takecare of sonarcube we need to add webhook with sonarcube url

or
To enforce Quality Gates in Jenkins, we configure a webhook in SonarQube that calls back to Jenkins with the analysis result.

Without the webhook, Jenkins won’t know when SonarQube analysis is complete. Using this setup along with waitForQualityGate(), we ensure the pipeline pauses and only proceeds if the code meets quality standards.

Integrated SonarQube as a Static Code Analysis (SAST) tool into Jenkins CI/CD pipelines, enforcing Quality Gates with webhook-based feedback on new code (bugs, vulnerabilities, and coverage ≥80%), enabling a Shift-Left DevSecOps approach and significantly reducing technical debt while improving code security and maintainability.

### Weekly Remediation Plan

* **Week 1:** Resolve *Code Issues*
* **Week 2:** Fix *Vulnerabilities*
* **Week 3:** Address *Bugs*

SonarQube Quality Gates
🔴 Bugs What it means: Definite coding errors that will cause the program to behave incorrectly at runtime.

🟠 Code Smells What it means: Not actual errors, but poor coding practices that make code hard to maintain or read.

🟡 Coverage What it means: Percentage of code covered by automated tests (unit/integration tests).

🟣 Issues What it means: The total number of all problems detected = Bugs + Code Smells + Vulnerabilities.

🟢 Maintainability Rating What it means: A grade (A–E) showing how easy it is to maintain the code.

🔐 Security Rating What it means: Grade (A–E) for how secure the code is.

🛡️ Vulnerabilities What it means: Parts of the code that could be exploited by attackers.
That’s a great achievement 👏 Let me put your experience into a story format (like you would tell in an interview, or write in your resume/LinkedIn post).

🚀 My SonarQube Implementation Journey

When I joined the project, we had no SonarQube integration in place. Code quality, coverage, and vulnerabilities were not being measured at all. I was assigned the responsibility of onboarding all project components into SonarQube and ensuring developers adopted the tool effectively.

Since it was a major shift, I decided to roll this out in stages over 3–4 months:

Phase 1 – Initial Setup (Month 1)

Configured SonarQube server and integrated it with Jenkins pipelines.

Added SonarQube scanner plugin and server configuration in Jenkins.

Created SonarQube projects and requested each team to add sonar-project.properties in the root of their repositories.

Ran the first scans to generate baseline reports of bugs, code smells, vulnerabilities, maintainability, and coverage.

Purpose: Give visibility to developers about the current state of their code without blocking them.

Phase 2 – Developer Adoption (Months 2–3)

Educated developers on how to interpret SonarQube reports.

Set expectations: they had 2.5 months to fix issues (bugs, vulnerabilities, code smells, maintainability, coverage).

Held weekly tracking sessions to monitor progress and help teams clear blockers.

Encouraged best practices (unit testing, removing duplications, refactoring long methods, fixing security flaws).

Phase 3 – Enforcing Quality Gates (Month 3–4)

Introduced SonarQube Quality Gates in the CI pipeline.

Initially configured the pipeline to show warnings but not fail builds.

Gave developers 2 weeks buffer to adapt to the gate requirements.

After 2 weeks, enabled strict enforcement: if the code failed the quality gate (bugs, coverage < threshold, security issues, etc.), the pipeline was aborted automatically.

Outcome

Within 3–4 months, we achieved:

Consistent code quality checks across all project components.

Higher unit test coverage (teams motivated to reach at least 80%).

Reduced vulnerabilities and better security posture.

Improved maintainability of codebase.

Developers became self-reliant in using SonarQube reports to improve their code.

Because of the impact on code quality and team adoption, I was recognized with the Best Performer Award in the project.

✨ Key Skills Demonstrated

CI/CD pipeline integration (Jenkins + SonarQube).

Change management (gradual adoption instead of forcing developers from day one).

Tracking and enforcing quality over time.

Collaboration with developers, ensuring code quality culture.

Security and maintainability improvements in large-scale codebases.

# Open Source Library Scan

We have **Nexus IQ**, an enterprise-level scanning tool. In an interview, we can mention that we recently **decommissioned** it and started using **GitHub Dependabot**.

## Enable Dependabot

For the `catalogue` repository:

**Settings → Dependabot → Enable**

Dependabot reads the `package.json` file and finds vulnerabilities in the dependencies.

We enabled **Dependabot** in all our repositories, and we check the **Dependabot alerts** in our pipeline.

If we see **High** or **Critical** alerts, we fail the pipeline.

## Get Dependabot Alerts Using GitHub API

Run the following command in Bash:

```bash id="j5w2nq"
curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/nagalakshmi477/catalogue/dependabot/alerts
```

### Create a GitHub Token

Go to:

**Settings → Developer Settings → Personal Access Tokens → Create Token**

Use the generated token in place of `YOUR_TOKEN`.

> Run the command in **Bash**.


# Dynamic Application Security Testing

DAST attacks the **running application** and performs security testing in the **UAT environment**.

We use **Veracode**, a third-party security testing tool.

# Jenkins Shared Library

**Jenkins Shared Pipeline**, also called a **Shared Library**, is used to **reuse common pipeline code** across multiple Jenkins jobs.

Instead of writing the same pipeline logic in every `Jenkinsfile`, we define reusable code in a **Shared Library** and call it from different pipelines.

We need to configure the **centralized pipeline code** in Jenkins.

**Manage Jenkins → System → Global Pipeline Libraries**


# ARGO CD

**Argo CD** is a deployment tool used to deploy applications on **Kubernetes (K8s)** and manage the cluster.

### Main Responsibilities

1. Maintain the cluster.
2. Deploy applications.

# How It Works

```text
Jenkins
(AWS Auth + kubectl)
        |
        | Deploy
        ↓
   EKS Cluster
```

* We need to deploy **Jenkins** into the **EKS cluster**.
* In Jenkins, we need to install **kubectl** and configure **AWS authentication**.
* Now Jenkins can connect to **EKS** and deploy the application.

## GitOps

**GitOps** means having a **single source of truth**. Everything should be maintained inside **Git**.

Instead of someone manually applying changes or running commands, GitOps means that when something is changed in the Git repository, it should automatically sync with the cluster.

### Example

If we change the image version from `1.0.0` to `1.0.1`, instead of manually running a Helm command, **Argo CD** looks at the Git repository and automatically detects and syncs the changes.

## Argo CD Inside the Cluster

Argo CD components are installed **inside the Kubernetes cluster**.


# Advantages

1. No need to install extra tools inside Jenkins, such as `kubectl`.
2. No need to provide **EKS authentication** to Jenkins.
3. **GitOps** → Git acts as a **single source of truth**. We don't need to apply changes manually; Argo CD will automatically sync the changes.
4. To revert to an old version, change the image version back in `values.yaml`. It will automatically restore the previous version, or we can raise a PR to revert to the previous image version.
5. **Argo CD** is also used for **cluster management**.

# Create Namespace

```bash id="q8nd7v"
kubectl create namespace argocd
```

# Install Argo CD

```bash id="8u2hlz"
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

kubectl get pods -n argocd

kubectl port-forward svc/argocd-server -n argocd 8080:443
```

# Argo CD Workflow

**Argo CD Source → Git Repository**

**Argo CD Destination → EKS Cluster**

Argo CD continuously checks the Git repository for changes. When it detects a change, it automatically applies those changes to the **EKS cluster**.


# Monitoring

## White Box Monitoring

**White box monitoring** means monitoring the inside of a system by checking **logs, metrics, and internal system information**.

## Black Box Monitoring

**Black box monitoring** means monitoring the system from an **end-user perspective**.

# RCA Analysis

**RCA** stands for **Root Cause Analysis**.

Incidents can be categorized as:

**P0, P1, P2, P3, P4, ...**

* **P0** means the system is completely down.
* Every **alert** is treated as an **incident**.

# 4 Golden Signals

## 1. Latency

How quickly our system is responding to requests.

* Lower latency is better.

## 2. Traffic

How many requests the system is receiving.

* For example, requests per minute or requests per hour.

## 3. Errors

The number of failed requests.

* We can monitor errors using **HTTP status codes**.

## 4. Saturation

How much of the system's resources are being utilized.

Examples:

* CPU
* Memory
* Disk
* Network

# Prometheus

**Prometheus** is a **time-series database and monitoring system**.

**TSD (Time-Series Database)** stores metric values against time.

```text
              Pull / Scrape
                   |
                   v
Prometheus --------------------> Node Exporter
     |
     | Stores Metrics
     v
    TSD
```

## Node Exporter

**Node Exporter** is installed on the server or node. It collects infrastructure-level metrics such as:

* CPU
* Memory
* Disk
* Network usage

Node Exporter exposes these metrics on **port 9100**.

**Prometheus** periodically scrapes this endpoint and pulls the metrics.

Prometheus then stores these time-series metrics in its local storage or in a configured remote time-series database.

> **Important:** Node Exporter does not push metrics to Prometheus. Prometheus pulls (scrapes) the metrics from Node Exporter.
