# catalogue

create ROBOSHOP ---> floder --> create ---> save
ROBOSHOP --> create pipeline 

here versions are changing so we need read the version from package.json file
for this we need to install one pluggin inside json (pipeline utility steps) --> pluggin

Depending installations   

we need login into agent and run the 
dnf module disable nodejs
sudo dnf module enable nodejs:20 -y
dnf install nodejs -y

create a webhook ---> configure --> add --> github hook 

now we need to create docker image
docker build -t URL/catalogue:appVersion

now we need to push to ECR

AWS --> Amzon ECR --> private registry ---> repo --> create --> roboshop/catalogue --> mutable --> create
there is a instruction tab it will show all process
1. we need to login for this we need to crate credentails

pluggin --> aws credentails 

manage jenkins ---> credenatils ---> kind --> aws credentails --> aws-auth(id)--> ssh-cre --> acess key and secreat key 
pluggin --> aws steps
this for api hitting 


install docker plugin on jenkins agent

sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/rhel/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user

disconnect and connect agent 

run pipeline --> we can see image on ECR --> scan 

testing --> functional testing -->DEV --> devlopers/testers
            integration testing --> UAT --> here all components needs to communicate properly


now image is ready we need to deploy 

Create INFRA floder
-----------------
install terraform on node
Install required utilities

sudo dnf install -y dnf-utils yum-utils

Add HashiCorp repository

sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo

Install Terraform

sudo dnf install -y terraform

Verify installation

terraform -version

create node --00-vpc --> pipeline

configure -- 00-vpc/jenkins 

pluggin ---> ansi clour

If any pipeline fail after installing plugins we need to restart the jenkins
sudo systemctl restart jenkins


UPSTREAM and DOWN STREAM
======================

create pipeline for sg 
pipleline --10-sg
down copy from 00-vpc --> Same process 


catalogue --> For CI
catalogue-cd --> for CD
on node side we cam install kubectl
curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.33.10/2026-04-08/bin/linux/amd64/kubectl
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client

cerate pipline for catalogue-ci and catalgue-cd on roboshop floder

HEre vpc is peering is required
install rebuilder pluggin for rebuld

we will get an error
 err="couldn't get current server API group list: Get \"https://104097364FA6AA759BD576D04FDCF3C2.gr7.us-east-1.eks.amazonaws.com/api?timeout=32s\": dial tcp 10.0.11.18:443: i/o timeout"

 so for this we need to something manually
 ec2 --> 
 eks --> networking --> cluster sg --> it ill go to vpc
 vpc --> sg --> roboshop-dev-coyntrolplain ---> edit inbound rules ---> 
 add rule --> https --> agent(sg_id) copy --> sg_id
 then we will get sucess

 for catalogue mongodb needs to running
 1. create infra
 2. steup eks setup
 3. we can deploy application


 1. first time failure, helm can't rollback beacuse there will be no version(0 version)
 2. 2nd deployemenet also failure, helm rollback attempts rollback also failure
    deployment failed rollback failed
3. 3rd deployement sucess ,3rd deployement sucess 
4.4th revisoin failure rollback is sucess deployment is failure

1st Deployment
    |
    FAILED
    |
    No previous revision
    |
    Rollback NOT possible
    |
    Deployment Failed


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


3rd Deployment
    |
    SUCCESS
    |
    Deployment Successful


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

Scans
=========
shift left --> brining testing and scaning to the early stages like DEV
instead of doing in higher env

build onces in DEV and run anywhere --> we only build the application in dev environmenet, we don't build in multiple environement we promote the application to multiple envi with diff configuration

static source code analysis ---> sonarcube
static application security testing --> sonarcube,fortify scan, github
open source library scan --> nexus iq, github dependabot
dynamic application security testing --> attacks on running application .. fortify webinspect, veracode 
docker image scan ---> vulnerabilities in the images

for sonarcude the instalation and setup was diffecut so we will use aws maretplace sonarcue

aws --> ec2--> sonarqube ce on aws and create instance

for sonarcube 9000 port number

jenkins serber --(we have source code --> someone should anayale it (scanner) --> scanner agnet --> will send the results to --> scanner server analyse results --> after alanyse) the it will give response to jenkins
based on the quality gates we ceck the results
scanner agenet will scees the code it will analyse the code the result will be moved to centaral server
on server scaner actall analyse will happen

What is correct in your diagram
Jenkins triggers the process ✔️
Scanner agent runs inside Jenkins ✔️
Agent scans the code ✔️
Results are sent to the scanning server ✔️
Server does the actual analysis ✔️
Quality Gates are involved ✔️

🔧 Small improvement (important)

Right now your diagram shows:

Scanner agent → results → server → analysis
But missing one step visually

👉 After server analysis:

Server sends result back to Jenkins
Then Jenkins checks Quality Gate status

Jenkins
   ↓
Scanner Agent (scans code)
   ↓
Send results → Scanner Server
   ↓
Server does full analysis
   ↓
Apply Quality Gates
   ↓
Send status back to Jenkins
   ↓
Jenkins decides (Pass ✅ / Fail ❌)

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
   → Compute metrics (bugs, vulnerabilities, code smells)
            ↓
   [Quality Gates Evaluation]
   → Pass / Fail based on conditions
            ↓
   [Response Sent Back to Jenkins]
            ↓
   [Jenkins Pipeline Decision]
   → If PASS → Continue build/deploy 🚀
   → If FAIL → Stop pipeline ❌


we need to install pluggin ---> sonarqube scanner --> It will enable sonarqube ptions

manage jenkis --> tools --> name(sonar-7.0) --> install automatically

agnet needs to where the scanning server is present
we need to give url in jenkins ---> manage --> system --> sonarcube servers --> add sonarcube --> sonar-7.2 --->http://<sonar_ip_pub>:9000 --> add token using sonarcube --> create
now we will add one stage in cI

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

Open source library scan
=============================
we have Nexus IQ --> enterprise level scanning tool . in interview we can mention that we recently decommissined this and started using github dependabot
catalogue ---> setting ---> depanboot --> enable
it reads package.json and findout the issues

we enabled dependabot in all our repos, we are checking the dependaboot alterts in our pipeline
if we see high and critical alters we are failing the pipeline

curl -L \
  -H "Accept: application/vnd.github+json" \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "X-GitHub-Api-Version: 2022-11-28" \
  https://api.github.com/repos/nagalakshmi477/catalogue/dependabot/alerts

for token --> setting --> develooper setting --> personal --> token 
create token

run in bash