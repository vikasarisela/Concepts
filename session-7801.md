# Enterprise CI/CD Pipeline

## GitHub + Jenkins + Dependabot + SonarQube + Docker + AWS ECR

---

# 1. Project Overview

This project demonstrates an enterprise-style CI/CD pipeline.

The source code is stored in **GitHub**.

**Jenkins** automatically builds and validates the application.

The pipeline performs:

1. Source-code checkout
2. Node.js dependency installation
3. Unit testing
4. Code coverage generation
5. Dependency vulnerability checking
6. SonarQube code-quality/security analysis
7. SonarQube Quality Gate validation
8. Docker image creation
9. AWS ECR authentication
10. Docker image push to Amazon ECR

## Overall flow

```text
Developer
    |
    | git push
    v
GitHub Repository
    |
    +--------------------+
    |                    |
    v                    v
Dependabot             Jenkins
    |                    |
    |                    v
    |              Checkout Code
    |                    |
    |                    v
    |              npm install
    |                    |
    |                    v
    |              npm test
    |                    |
    |                    v
    |            Coverage Report
    |                    |
    |                    v
    |              SonarQube
    |                    |
    |                    v
    |              Quality Gate
    |                    |
    |                    v
    |              Docker Build
    |                    |
    |                    v
    |                AWS ECR
    |                    |
    |                    v
    |              Push Image
    |
    v
Security alerts / PRs
```

---

# 2. Technologies Used

| Technology        | Purpose                            |
| ----------------- | ---------------------------------- |
| Git               | Source-code version control        |
| GitHub            | Remote source-code repository      |
| GitHub Dependabot | Dependency vulnerability detection |
| Jenkins           | CI/CD automation                   |
| Node.js           | Application runtime                |
| npm               | Node.js dependency management      |
| Jest              | Unit testing                       |
| SonarQube         | Code quality and security analysis |
| Docker            | Application containerization       |
| AWS ECR           | Docker image registry              |
| AWS IAM           | AWS authentication/authorization   |

---

# 3. Repository Structure

The repository can look like this:

```text
enterprise-pipeline/
│
├── .github/
│   └── dependabot.yml
│
├── src/
│   └── app.js
│
├── tests/
│   └── app.test.js
│
├── package.json
├── package-lock.json
├── Dockerfile
├── sonar-project.properties
└── Jenkinsfile
```

---

# 4. Application Source Code

## src/app.js

This is a simple Node.js application.

```javascript
function add(a, b) {
    return a + b;
}

function multiply(a, b) {
    return a * b;
}

module.exports = {
    add,
    multiply
};
```

### Explanation

The application contains two functions:

```javascript
add()
```

Adds two numbers.

```javascript
multiply()
```

Multiplies two numbers.

The functions are exported so that Jest can test them.

---

# 5. package.json

```json
{
  "name": "enterprise-pipeline",
  "version": "1.0.0",
  "description": "Enterprise CI/CD pipeline demonstration",
  "main": "src/app.js",
  "scripts": {
    "test": "jest --coverage"
  },
  "dependencies": {
    "express": "^4.21.2"
  },
  "devDependencies": {
    "jest": "^29.7.0"
  }
}
```

---

# 6. What is package.json?

`package.json` defines:

* Application name
* Application version
* Dependencies
* Development dependencies
* npm commands

For example:

```json
"scripts": {
    "test": "jest --coverage"
}
```

means when Jenkins executes:

```bash
npm test
```

npm actually runs:

```bash
jest --coverage
```

---

# 7. package-lock.json

When you execute:

```bash
npm install
```

npm normally creates or updates:

```text
package-lock.json
```

The lock file records the resolved dependency versions.

For example:

```text
package.json
      |
      v
Requested dependency
      |
      v
package-lock.json
      |
      v
Exact resolved dependency tree
```

The lock file helps ensure reproducible installations.

---

# 8. Important Difference: npm install vs Updating a Version

This was an important point in our earlier discussion.

If you already changed:

```json
"express": "^4.21.2"
```

to:

```json
"express": "^4.21.3"
```

then:

```bash
npm install
```

reads the updated `package.json` and installs/resolves the dependency.

`npm install` itself does **not necessarily mean "upgrade everything."**

It installs dependencies according to the dependency definitions and lock file.

---

# 9. Dependabot

Create:

```text
.github/dependabot.yml
```

Source:

```yaml
version: 2

updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
```

---

# 10. What Dependabot Does

Dependabot monitors dependencies.

For example:

```text
package.json

express: 4.21.2
```

Suppose GitHub determines that this version has a known vulnerability.

Dependabot can identify the vulnerable dependency.

It may create an alert such as:

```text
Dependency:
express

Current version:
4.21.2

Vulnerability:
Security vulnerability

Recommended:
Upgrade to a newer version
```

Depending on repository configuration, Dependabot may also create a pull request.

---

# 11. Important Dependabot Point

Dependabot is **not the same thing as npm install**.

Dependabot:

```text
Monitors dependencies
        |
        v
Identifies known vulnerabilities
        |
        v
Suggests/creates updates
```

Jenkins:

```text
Checks out source
        |
        v
Installs dependencies
        |
        v
Tests application
        |
        v
Analyzes code
        |
        v
Builds Docker image
```

---

# 12. tests/app.test.js

```javascript
const { add, multiply } = require("../src/app");

describe("Application tests", () => {

    test("adds two numbers", () => {
        expect(add(2, 3)).toBe(5);
    });

    test("multiplies two numbers", () => {
        expect(multiply(2, 3)).toBe(6);
    });

});
```

---

# 13. What Type of Testing Is This?

This is:

```text
Unit Testing
```

Jest tests individual application functions.

Example:

```javascript
add(2, 3)
```

Expected result:

```text
5
```

This is not:

* Integration testing
* System testing
* Performance testing
* UI testing

It is primarily **unit testing**.

---

# 14. Code Coverage

When we run:

```bash
npm test
```

the package script executes:

```bash
jest --coverage
```

Jest generates coverage information.

Typical output:

```text
Coverage summary

Statements   : 100%
Branches     : 100%
Functions    : 100%
Lines        : 100%
```

Coverage answers:

> How much of our code was executed by the tests?

It does **not** mean:

> The application has no bugs.

---

# 15. SonarQube

SonarQube performs static analysis.

It can identify things such as:

* Bugs
* Vulnerabilities
* Code smells
* Duplicated code
* Maintainability issues
* Security issues
* Test coverage information

The basic flow is:

```text
Source Code
    |
    v
SonarScanner
    |
    v
SonarQube Server
    |
    v
Analysis
    |
    v
Quality Gate
```

---

# 16. sonar-project.properties

Create:

```text
sonar-project.properties
```

Example:

```properties
sonar.projectKey=enterprise-pipeline
sonar.projectName=enterprise-pipeline

sonar.sources=src
sonar.tests=tests

sonar.javascript.lcov.reportPaths=coverage/lcov.info

sonar.exclusions=node_modules/**
```

### Explanation

```properties
sonar.projectKey=enterprise-pipeline
```

Unique project identifier in SonarQube.

```properties
sonar.sources=src
```

Tells SonarQube where application source code exists.

```properties
sonar.tests=tests
```

Tells SonarQube where tests are located.

```properties
sonar.javascript.lcov.reportPaths=coverage/lcov.info
```

Allows SonarQube to read the Jest coverage report.

---

# 17. Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install --omit=dev

COPY src ./src

EXPOSE 3000

CMD ["node", "src/app.js"]
```

---

# 18. Dockerfile Explanation

## FROM

```dockerfile
FROM node:20-alpine
```

Uses a Node.js base image.

---

## WORKDIR

```dockerfile
WORKDIR /app
```

Sets:

```text
/app
```

as the working directory inside the container.

---

## COPY package files

```dockerfile
COPY package*.json ./
```

Copies:

```text
package.json
package-lock.json
```

if available.

---

## Install production dependencies

```dockerfile
RUN npm install --omit=dev
```

Installs dependencies required by the application while excluding development dependencies.

---

## Copy application

```dockerfile
COPY src ./src
```

Copies source code into the container.

---

## EXPOSE

```dockerfile
EXPOSE 3000
```

Documents the application's expected port.

---

## CMD

```dockerfile
CMD ["node", "src/app.js"]
```

Starts the application.

---

# 19. Jenkins Agent Requirements

The Jenkins agent executing this pipeline should have the required tools.

At minimum:

```text
Java
Git
Node.js
npm
Docker
AWS CLI
SonarScanner
curl
```

Check them with:

```bash
java -version
```

```bash
git --version
```

```bash
node --version
```

```bash
npm --version
```

```bash
docker --version
```

```bash
aws --version
```

```bash
sonar-scanner --version
```

---

# 20. Why These Tools Must Be on the Agent

If your Jenkins pipeline uses:

```groovy
agent {
    label 'agent-1'
}
```

then Jenkins executes the stages on the node labeled:

```text
agent-1
```

Therefore, commands such as:

```bash
npm install
docker build
aws ecr get-login-password
sonar-scanner
```

must be available on that agent.

Jenkins controller does not automatically provide these commands.

---

# 21. Jenkins Credentials

The pipeline needs authentication to external systems.

Typical credentials:

```text
Jenkins
 |
 +-- GitHub credentials
 |
 +-- SonarQube token
 |
 +-- AWS credentials / IAM role
 |
 +-- ECR access
```

---

# 22. GitHub Token

The GitHub token allows Jenkins to communicate with GitHub when required.

Example permissions depend on the exact operation.

For private repositories, Jenkins normally needs repository read access.

For GitHub API operations such as Dependabot alerts, additional permissions may be required.

---

# 23. SonarQube Token

Jenkins needs authentication to send analysis to SonarQube.

Example Jenkins credential:

```text
Credential ID:
sonarqube-token
```

The token should **not** be hard-coded in the Jenkinsfile.

Bad:

```groovy
SONAR_TOKEN = 'actual-secret-token'
```

Good:

```groovy
credentials('sonarqube-token')
```

---

# 24. AWS Authentication

Jenkins needs AWS permissions to push images to ECR.

The preferred approach in AWS environments is usually:

```text
Jenkins Agent
      |
      v
IAM Role
      |
      v
AWS ECR
```

If using Jenkins-stored AWS credentials, they must have appropriate ECR permissions.

---

# 25. Example ECR Permissions

Typical ECR push permissions include:

```text
ecr:GetAuthorizationToken
ecr:BatchCheckLayerAvailability
ecr:InitiateLayerUpload
ecr:UploadLayerPart
ecr:CompleteLayerUpload
ecr:PutImage
```

Use least privilege rather than giving unrestricted AWS permissions.

---

# 26. Complete Jenkinsfile

```groovy
pipeline {

    agent {
        label 'agent-1'
    }

    environment {

        // AWS configuration
        AWS_ACCOUNT_ID = '193849563622'
        AWS_REGION     = 'us-east-1'
        ECR_REPO_NAME  = 'enterprise-pipeline'

        // GitHub configuration
        GITHUB_ORG     = 'vikasarisela'
        GITHUB_REPO    = 'enterprise-pipeline'

        // Application configuration
        APP_NAME       = 'enterprise-pipeline'
        IMAGE_TAG      = "${BUILD_NUMBER}"

        // SonarQube project
        SONAR_PROJECT_KEY = 'enterprise-pipeline'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code from GitHub'

                checkout scm
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                    echo "Checking installed tools..."

                    git --version
                    node --version
                    npm --version
                    docker --version
                    aws --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                echo 'Installing Node.js dependencies'

                sh '''
                    npm install
                '''
            }
        }

        stage('Unit Test & Coverage') {
            steps {
                echo 'Running unit tests and generating coverage'

                sh '''
                    npm test
                '''
            }

            post {
                always {
                    archiveArtifacts artifacts: 'coverage/**',
                                     allowEmptyArchive: true
                }
            }
        }

        stage('Dependency Security Check') {
            steps {
                echo 'Checking npm dependencies for known vulnerabilities'

                sh '''
                    npm audit --audit-level=high
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo 'Running SonarQube analysis'

                withSonarQubeEnv('sonarqube') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.sources=src \
                          -Dsonar.tests=tests \
                          -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                          -Dsonar.exclusions=node_modules/**
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {
                echo 'Waiting for SonarQube Quality Gate'

                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo 'Building Docker image'

                sh '''
                    docker build \
                      -t ${APP_NAME}:${IMAGE_TAG} \
                      .
                '''
            }
        }

        stage('ECR Login') {
            steps {
                echo 'Logging in to Amazon ECR'

                sh '''
                    aws ecr get-login-password \
                      --region ${AWS_REGION} \
                    | docker login \
                      --username AWS \
                      --password-stdin \
                      ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {
                echo 'Tagging Docker image for ECR'

                sh '''
                    docker tag \
                      ${APP_NAME}:${IMAGE_TAG} \
                      ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {
                echo 'Pushing Docker image to Amazon ECR'

                sh '''
                    docker push \
                      ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }

        always {
            echo 'Pipeline execution completed'
        }
    }
}
```

---

# 27. Jenkinsfile Stage-by-Stage Explanation

## Stage 1 — Checkout

```groovy
stage('Checkout') {
    steps {
        checkout scm
    }
}
```

Jenkins obtains the source code from GitHub.

Flow:

```text
GitHub
   |
   v
Jenkins
   |
   v
Workspace
```

---

# 28. Jenkins Workspace

Jenkins creates a workspace similar to:

```text
/home/ec2-user/jenkins/workspace/enterprise-pipeline
```

The repository is checked out there.

Commands execute inside this workspace.

---

# 29. Stage 2 — Verify Tools

```groovy
stage('Verify Tools')
```

Checks:

```bash
git --version
node --version
npm --version
docker --version
aws --version
```

This is useful because if a command is missing, we discover it early.

For example:

```text
java: command not found
```

means Java isn't available to the agent.

Similarly:

```text
docker: command not found
```

means Docker is not installed or isn't in PATH.

---

# 30. Stage 3 — Install Dependencies

```groovy
npm install
```

npm reads:

```text
package.json
```

and:

```text
package-lock.json
```

and installs dependencies into:

```text
node_modules/
```

After this stage:

```text
enterprise-pipeline/
├── node_modules/
├── package.json
└── package-lock.json
```

---

# 31. First Pipeline Run

If this is the first time the project is being built, `node_modules` may not exist.

Therefore:

```bash
npm install
```

creates:

```text
node_modules/
```

and installs the dependencies.

This is why the pipeline needs an installation step before testing.

---

# 32. Stage 4 — Unit Test

```bash
npm test
```

runs:

```bash
jest --coverage
```

Jest executes:

```text
tests/app.test.js
```

and tests:

```text
add()
multiply()
```

---

# 33. Coverage Output

Jest creates something similar to:

```text
coverage/
├── lcov.info
├── lcov-report/
└── coverage-summary.json
```

SonarQube can consume:

```text
coverage/lcov.info
```

---

# 34. Stage 5 — Dependency Security

```bash
npm audit --audit-level=high
```

This checks npm dependencies against known vulnerability information.

For example:

```text
package
   |
   v
Known vulnerable version?
   |
   +---- No ----> Continue
   |
   +---- Yes ---> Pipeline can fail
```

This is different from SonarQube.

---

# 35. Dependabot vs npm audit

They serve related but different purposes.

### Dependabot

Runs through GitHub and monitors dependency vulnerabilities/updates.

```text
GitHub
   |
   v
Dependabot
   |
   v
Dependency vulnerability
   |
   v
Alert / Pull Request
```

### npm audit

Runs during the Jenkins build.

```text
Jenkins
   |
   v
npm audit
   |
   v
Check dependencies
```

### SonarQube

Analyzes the source code.

```text
Source Code
    |
    v
SonarQube
    |
    v
Bugs / Vulnerabilities / Code Smells / Coverage
```

---

# 36. Stage 6 — SonarQube Analysis

```groovy
withSonarQubeEnv('sonarqube')
```

This tells Jenkins to use the SonarQube server configuration named:

```text
sonarqube
```

Then:

```bash
sonar-scanner
```

sends analysis information to SonarQube.

---

# 37. SonarQube Connection

The connection is:

```text
Jenkins
   |
   | SonarScanner
   |
   v
SonarQube Server
   |
   v
Project Analysis
```

The authentication token is stored securely in Jenkins/SonarQube configuration.

Do not put the token directly into GitHub.

---

# 38. Stage 7 — Quality Gate

```groovy
waitForQualityGate abortPipeline: true
```

This is important.

SonarQube calculates the Quality Gate.

Example:

```text
SonarQube Analysis
       |
       v
Quality Gate
       |
   +---+---+
   |       |
 PASS    FAIL
   |       |
   v       v
Continue  Stop
```

If the Quality Gate fails:

```text
Pipeline stops
```

because:

```groovy
abortPipeline: true
```

---

# 39. Stage 8 — Docker Build

```bash
docker build -t enterprise-pipeline:BUILD_NUMBER .
```

Docker reads:

```text
Dockerfile
```

and creates an image.

Example:

```text
Dockerfile
     |
     v
docker build
     |
     v
Docker Image
```

---

# 40. Why `.` Is Used

This command:

```bash
docker build -t enterprise-pipeline:10 .
```

has:

```text
. 
```

at the end.

The `.` means:

> Use the current directory as the Docker build context.

Without it, Docker can return an error such as:

```text
docker buildx build requires 1 argument
```

---

# 41. Docker Image Tag

The pipeline uses:

```groovy
IMAGE_TAG = "${BUILD_NUMBER}"
```

If Jenkins build number is:

```text
25
```

then:

```text
IMAGE_TAG=25
```

The image becomes:

```text
enterprise-pipeline:25
```

This provides a unique tag for each Jenkins build.

---

# 42. Stage 9 — ECR Login

Command:

```bash
aws ecr get-login-password \
  --region ${AWS_REGION} \
| docker login \
  --username AWS \
  --password-stdin \
  ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
```

Flow:

```text
Jenkins Agent
      |
      v
AWS CLI
      |
      v
AWS ECR authentication
      |
      v
Docker login
```

---

# 43. AWS Account ID

Example:

```groovy
AWS_ACCOUNT_ID = '193849563622'
```

Your actual AWS account ID must be used.

Do not blindly use the example account ID.

---

# 44. ECR Repository

Example:

```groovy
ECR_REPO_NAME = 'enterprise-pipeline'
```

ECR should contain a repository named:

```text
enterprise-pipeline
```

The full ECR address becomes:

```text
ACCOUNT_ID.dkr.ecr.REGION.amazonaws.com/REPOSITORY
```

Example:

```text
193849563622.dkr.ecr.us-east-1.amazonaws.com/enterprise-pipeline
```

---

# 45. Stage 10 — Docker Tagging

Initially:

```text
enterprise-pipeline:25
```

Then:

```bash
docker tag \
enterprise-pipeline:25 \
193849563622.dkr.ecr.us-east-1.amazonaws.com/enterprise-pipeline:25
```

Now Docker knows where the image will be pushed.

---

# 46. Stage 11 — Docker Push

```bash
docker push \
193849563622.dkr.ecr.us-east-1.amazonaws.com/enterprise-pipeline:25
```

Flow:

```text
Jenkins Agent
      |
      v
Docker Image
      |
      v
AWS ECR
```

---

# 47. Final ECR Result

AWS ECR contains:

```text
enterprise-pipeline
│
├── 1
├── 2
├── 3
├── 4
└── 25
```

Each tag represents a Jenkins build.

For example:

```text
Build 25
    |
    v
Docker image
    |
    v
ECR tag 25
```

---

# 48. Complete End-to-End Flow

A developer changes the application.

```bash
git add .
git commit -m "Update application"
git push origin main
```

Then:

```text
GitHub
   |
   v
Jenkins Trigger
   |
   v
Checkout
   |
   v
npm install
   |
   v
npm test
   |
   v
Coverage
   |
   v
npm audit
   |
   v
SonarQube
   |
   v
Quality Gate
   |
   v
Docker Build
   |
   v
ECR Login
   |
   v
Docker Tag
   |
   v
Docker Push
   |
   v
AWS ECR
```

---

# 49. Where Dependabot Fits

Dependabot works alongside the CI/CD pipeline.

```text
                    GitHub
                   /      \
                  /        \
                 v          v
          Dependabot      Jenkins
              |              |
              v              v
       Vulnerability      Build/Test
          Alerts              |
              |               v
              |          SonarQube
              |               |
              |               v
              |          Docker Build
              |               |
              |               v
              |             ECR
              |
              v
       Dependency Update
              |
              v
          Pull Request
```

---

# 50. Example Vulnerability Scenario

Suppose:

```json
"express": "^4.21.2"
```

has a known vulnerability.

Dependabot detects it.

```text
Dependabot
     |
     v
Vulnerability detected
     |
     v
GitHub alert
     |
     v
Recommended version
```

You update:

```json
"express": "^4.21.3"
```

Then run:

```bash
npm install
```

This updates the dependency tree and potentially:

```text
package-lock.json
```

Then:

```bash
git add package.json package-lock.json
git commit -m "Update express dependency"
git push origin main
```

Jenkins runs again.

---

# 51. Important Security Principle

Never store secrets directly in:

```text
Jenkinsfile
package.json
Dockerfile
GitHub repository
```

Bad:

```groovy
SONAR_TOKEN = "abc123secret"
```

Bad:

```bash
aws_access_key_id=XXXXXXXX
```

Instead use:

```text
Jenkins Credentials
AWS IAM Role
GitHub Secrets
SonarQube Credentials
```

---

# 52. Jenkins Credential Relationship

Think of the connections like this:

```text
GitHub
  |
  | Git credentials/token
  v
Jenkins
  |
  | SonarQube token
  v
SonarQube

Jenkins
  |
  | AWS IAM credentials/role
  v
AWS

AWS
  |
  v
ECR
```

The credentials are the authentication mechanism.

The permissions determine what the authenticated identity is allowed to do.

---

# 53. Jenkins Configuration

Create a Jenkins Pipeline job.

Example:

```text
Jenkins
   |
   v
New Item
   |
   v
Pipeline
```

Configure:

```text
Pipeline Definition:
Pipeline script from SCM
```

SCM:

```text
Git
```

Repository:

```text
Your GitHub repository
```

Branch:

```text
*/main
```

Script Path:

```text
Jenkinsfile
```

---

# 54. GitHub Repository

The repository should contain:

```text
enterprise-pipeline/
│
├── .github/
│   └── dependabot.yml
│
├── src/
│   └── app.js
│
├── tests/
│   └── app.test.js
│
├── package.json
├── package-lock.json
├── Dockerfile
├── sonar-project.properties
└── Jenkinsfile
```

---

# 55. Git Commands

Initialize:

```bash
git init
```

Add remote:

```bash
git remote add origin https://github.com/YOUR_USERNAME/enterprise-pipeline.git
```

Check:

```bash
git remote -v
```

Add files:

```bash
git add .
```

Commit:

```bash
git commit -m "Add enterprise CI/CD pipeline"
```

Push:

```bash
git push origin main
```

---

# 56. Correct Git Push Syntax

The command is:

```bash
git push origin main
```

Not:

```bash
git push orign main
```

`origin` is the default remote name.

---

# 57. Common Error — Dependabot Disabled

If Jenkins executes a GitHub API request and receives:

```text
Dependabot alerts are disabled for this repository.
```

that means GitHub's Dependabot alerts are not enabled for that repository.

This is not necessarily a Jenkins problem.

Check the GitHub repository's security/dependency settings and enable the required Dependabot features.

---

# 58. Common Error — Docker Permission

Error:

```text
permission denied while trying to connect to the Docker daemon socket
```

This usually means the Jenkins agent user cannot access Docker.

Check:

```bash
docker ps
```

as the Jenkins/agent user.

The agent user needs appropriate Docker access.

---

# 59. Common Error — Docker Build

Error:

```text
docker buildx build requires 1 argument
```

Usually check that the command contains the build context:

```bash
docker build -t myimage:tag .
```

The final:

```text
.
```

is important.

---

# 60. Common Error — Java Not Found

Error:

```text
java: command not found
```

This means Java isn't available to the Jenkins agent.

Check:

```bash
java -version
```

If missing, install/configure Java on the **agent that executes the pipeline**.

---

# 61. Important Jenkins Concept

This:

```groovy
agent {
    label 'agent-1'
}
```

means:

> Execute the pipeline on a Jenkins node having the `agent-1` label.

Therefore:

```text
Jenkins Controller
        |
        | assigns job
        v
agent-1
        |
        +-- git
        +-- node
        +-- npm
        +-- docker
        +-- aws
        +-- sonar-scanner
```

---

# 62. CI vs CD in This Project

## Continuous Integration

The following are primarily CI:

```text
Checkout
npm install
Unit Tests
Coverage
npm audit
SonarQube
Quality Gate
```

The goal is:

> Validate the source code before accepting it as a good build.

---

## Continuous Delivery / Deployment

These are closer to the delivery side:

```text
Docker Build
Docker Tag
Docker Push
```

The image is delivered to:

```text
AWS ECR
```

A later deployment pipeline could take the ECR image and deploy it to:

```text
ECS
EKS
EC2
Kubernetes
```

---

# 63. What This Pipeline Does NOT Do

This pipeline currently pushes the image to ECR.

It does not automatically deploy the container to:

```text
ECS
```

or:

```text
EKS
```

or:

```text
EC2
```

unless a deployment stage is added.

---

# 64. Final Architecture

```text
                  DEVELOPER
                      |
                      | git push
                      v
                  GITHUB
                 /      \
                /        \
               v          v
        DEPENDABOT      JENKINS
            |              |
            |              v
            |          CHECKOUT
            |              |
            |              v
            |         NPM INSTALL
            |              |
            |              v
            |          UNIT TEST
            |              |
            |              v
            |          COVERAGE
            |              |
            |              v
            |        NPM AUDIT
            |              |
            |              v
            |         SONARQUBE
            |              |
            |              v
            |        QUALITY GATE
            |              |
            |          PASS?
            |           /   \
            |         NO     YES
            |         |       |
            |       STOP       v
            |              DOCKER BUILD
            |                   |
            |                   v
            |              ECR LOGIN
            |                   |
            |                   v
            |              DOCKER TAG
            |                   |
            |                   v
            |              DOCKER PUSH
            |                   |
            |                   v
            |                AWS ECR
            |
            v
     SECURITY ALERT /
     DEPENDENCY UPDATE
```

---

# 65. One-Line Explanation of Every Component

### GitHub

Stores the source code.

### Git

Tracks source-code changes.

### Dependabot

Finds vulnerable/outdated dependencies and can propose updates.

### Jenkins

Automates the CI/CD process.

### npm

Installs Node.js dependencies and executes application scripts.

### Jest

Performs unit tests.

### Coverage

Measures how much application code was exercised by tests.

### SonarQube

Performs static code analysis and evaluates code quality/security.

### Quality Gate

Determines whether the code satisfies configured SonarQube quality conditions.

### Docker

Packages the application into a container image.

### AWS ECR

Stores Docker images.

### IAM

Controls AWS permissions.

---

# 66. Most Important Connection to Remember

Remember this:

```text
GitHub
   |
   | Source Code
   v
Jenkins
   |
   | Build/Test
   v
SonarQube
   |
   | Quality Gate
   v
Docker
   |
   | Container Image
   v
AWS ECR
```

And separately:

```text
GitHub
   |
   v
Dependabot
   |
   v
Dependency Vulnerability
   |
   v
Update package.json
   |
   v
GitHub
   |
   v
Jenkins
```

---

# 67. Interview Explanation

If an interviewer asks:

> Explain your CI/CD pipeline.

You can say:

> "Our source code is maintained in GitHub. When a developer pushes code, Jenkins checks out the repository and runs the CI pipeline. The pipeline installs Node.js dependencies, executes Jest unit tests and generates code coverage. It also performs an npm dependency security check. The source code and coverage information are then analyzed by SonarQube, and the pipeline waits for the SonarQube Quality Gate. If the quality gate passes, Jenkins builds a Docker image, authenticates with Amazon ECR using AWS credentials or an IAM role, tags the image with the Jenkins build number, and pushes it to ECR. Separately, GitHub Dependabot monitors project dependencies and identifies known vulnerabilities or proposes dependency updates."

---

# 68. Complete Project Files

## .github/dependabot.yml

```yaml
version: 2

updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
```

## src/app.js

```javascript
function add(a, b) {
    return a + b;
}

function multiply(a, b) {
    return a * b;
}

module.exports = {
    add,
    multiply
};
```

## tests/app.test.js

```javascript
const { add, multiply } = require("../src/app");

describe("Application tests", () => {

    test("adds two numbers", () => {
        expect(add(2, 3)).toBe(5);
    });

    test("multiplies two numbers", () => {
        expect(multiply(2, 3)).toBe(6);
    });

});
```

## package.json

```json
{
  "name": "enterprise-pipeline",
  "version": "1.0.0",
  "description": "Enterprise CI/CD pipeline demonstration",
  "main": "src/app.js",
  "scripts": {
    "test": "jest --coverage"
  },
  "dependencies": {
    "express": "^4.21.2"
  },
  "devDependencies": {
    "jest": "^29.7.0"
  }
}
```

## Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install --omit=dev

COPY src ./src

EXPOSE 3000

CMD ["node", "src/app.js"]
```

## sonar-project.properties

```properties
sonar.projectKey=enterprise-pipeline
sonar.projectName=enterprise-pipeline

sonar.sources=src
sonar.tests=tests

sonar.javascript.lcov.reportPaths=coverage/lcov.info

sonar.exclusions=node_modules/**
```

## Jenkinsfile

```groovy
pipeline {

    agent {
        label 'agent-1'
    }

    environment {

        AWS_ACCOUNT_ID = '193849563622'
        AWS_REGION     = 'us-east-1'
        ECR_REPO_NAME  = 'enterprise-pipeline'

        GITHUB_ORG     = 'vikasarisela'
        GITHUB_REPO    = 'enterprise-pipeline'

        APP_NAME       = 'enterprise-pipeline'
        IMAGE_TAG      = "${BUILD_NUMBER}"

        SONAR_PROJECT_KEY = 'enterprise-pipeline'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Verify Tools') {
            steps {
                sh '''
                    git --version
                    node --version
                    npm --version
                    docker --version
                    aws --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Unit Test & Coverage') {
            steps {
                sh 'npm test'
            }

            post {
                always {
                    archiveArtifacts artifacts: 'coverage/**',
                                     allowEmptyArchive: true
                }
            }
        }

        stage('Dependency Security Check') {
            steps {
                sh 'npm audit --audit-level=high'
            }
        }

        stage('SonarQube Analysis') {
            steps {

                withSonarQubeEnv('sonarqube') {

                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                          -Dsonar.sources=src \
                          -Dsonar.tests=tests \
                          -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
                          -Dsonar.exclusions=node_modules/**
                    '''
                }
            }
        }

        stage('Quality Gate') {
            steps {

                timeout(time: 5, unit: 'MINUTES') {

                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Docker Build') {
            steps {

                sh '''
                    docker build \
                      -t ${APP_NAME}:${IMAGE_TAG} \
                      .
                '''
            }
        }

        stage('ECR Login') {
            steps {

                sh '''
                    aws ecr get-login-password \
                      --region ${AWS_REGION} \
                    | docker login \
                      --username AWS \
                      --password-stdin \
                      ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com
                '''
            }
        }

        stage('Tag Docker Image') {
            steps {

                sh '''
                    docker tag \
                      ${APP_NAME}:${IMAGE_TAG} \
                      ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push Image to ECR') {
            steps {

                sh '''
                    docker push \
                      ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}
                '''
            }
        }
    }

    post {

        success {
            echo 'Pipeline completed successfully'
        }

        failure {
            echo 'Pipeline failed'
        }

        always {
            echo 'Pipeline execution completed'
        }
    }
}
```

---

# 69. Final Mental Model

The easiest way to remember the entire project is:

```text
             SOURCE
               |
             GitHub
               |
               v
             JENKINS
               |
       +-------+-------+
       |       |       |
       v       v       v
      npm     Jest   npm audit
    install    |       |
       |       v       |
       |    Coverage   |
       |       |       |
       +-------+-------+
               |
               v
           SonarQube
               |
               v
          Quality Gate
               |
             PASS
               |
               v
            Docker
               |
               v
          AWS ECR Login
               |
               v
          Docker Tag
               |
               v
          Docker Push
               |
               v
             ECR
```

**Security side:**

```text
GitHub
   |
   v
Dependabot
   |
   v
Dependency vulnerability
   |
   v
Update dependency
   |
   v
GitHub PR / commit
   |
   v
Jenkins pipeline again
```

This gives you the complete relationship between **source code, dependencies, testing, coverage, security, quality, containerization, and AWS image storage**.
