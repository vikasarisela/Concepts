# Enterprise CI/CD Pipeline – Jenkins + GitHub + SonarQube + Docker + AWS ECR

## 1. Overall Pipeline

The purpose of this pipeline is to automatically take application code from GitHub, test and scan it, create a Docker image, and store that image in AWS ECR.

```text
Developer
    |
    | git push
    v
GitHub
    |
    v
Jenkins
    |
    +--> 1. Checkout Code
    |
    +--> 2. Install Dependencies
    |
    +--> 3. Unit Test + Coverage
    |
    +--> 4. Dependency Security
    |
    +--> 5. SonarQube Analysis
    |
    +--> 6. Quality Gate
    |
    +--> 7. Docker Build
    |
    +--> 8. ECR Login
    |
    +--> 9. Docker Tag
    |
    +--> 10. Push Image
    |
    v
AWS ECR
```

**Remember:** Jenkins is the central automation tool. Each stage performs one validation or delivery activity.

---

# 2. Project Structure

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

# 3. Step 1 – Application Source Code

### Theory

First we need an application for Jenkins to build and test.

For this example, we have a simple Node.js application containing two functions.

### `src/app.js`

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

**Remember:** `src/` contains the actual application source code.

---

# 4. Step 2 – Define Dependencies

### Theory

Node.js applications normally require external packages.

`package.json` tells npm:

* What the application is
* Which packages are required
* Which commands should be executed

### `package.json`

```json
{
  "name": "enterprise-pipeline",
  "version": "1.0.0",
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

The important part for Jenkins is:

```json
"scripts": {
    "test": "jest --coverage"
}
```

Therefore:

```bash
npm test
```

actually executes:

```bash
jest --coverage
```

---

# 5. Step 3 – Install Dependencies

### Theory

After Jenkins downloads the source code, the agent needs the Node.js packages before the application can be tested.

Jenkins executes:

```bash
npm install
```

npm reads:

```text
package.json
     +
package-lock.json
     |
     v
node_modules/
```

The dependencies are installed into:

```text
node_modules/
```

### Jenkins stage

```groovy
stage('Install Dependencies') {
    steps {
        sh 'npm install'
    }
}
```

**Remember:** `npm install` prepares the environment required to run the application/tests.

---

# 6. Step 4 – Unit Testing + Coverage

### Theory

After installing dependencies, Jenkins needs to verify that the application behaves correctly.

We use **Jest** for unit testing.

A unit test checks a small individual piece of application logic.

### `tests/app.test.js`

```javascript
const { add, multiply } = require("../src/app");

test("adds two numbers", () => {
    expect(add(2, 3)).toBe(5);
});

test("multiplies two numbers", () => {
    expect(multiply(2, 3)).toBe(6);
});
```

Jenkins executes:

```bash
npm test
```

which runs:

```bash
jest --coverage
```

### Jenkins stage

```groovy
stage('Unit Test & Coverage') {
    steps {
        sh 'npm test'
    }
}
```

### Result

```text
Application Code
      |
      v
Jest
      |
      +--> Tests
      |
      +--> Coverage
```

**Remember:** Testing checks functionality. Coverage tells us how much of the code was executed by tests.

---

# 7. Step 5 – Dependency Security Check

### Theory

Even if our application code works, the third-party packages we use may contain known vulnerabilities.

Therefore Jenkins runs:

```bash
npm audit --audit-level=high
```

### Jenkins stage

```groovy
stage('Dependency Security') {
    steps {
        sh 'npm audit --audit-level=high'
    }
}
```

The flow is:

```text
package.json
     |
     v
Dependencies
     |
     v
npm audit
     |
     v
Known vulnerabilities?
     |
   +---+---+
   |       |
  No      Yes
   |       |
   v       v
Continue  Fail/Report
```

**Remember:** `npm audit` checks the dependencies during the Jenkins build.

---

# 8. Step 6 – Dependabot

### Theory

Dependabot works on the **GitHub side**.

It continuously checks dependencies for known vulnerabilities and available updates.

### `.github/dependabot.yml`

```yaml
version: 2

updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
```

Flow:

```text
GitHub
   |
   v
Dependabot
   |
   v
Check dependencies
   |
   v
Vulnerability / Update
   |
   v
Alert or Pull Request
```

**Important difference:**

```text
Dependabot
    |
    v
GitHub dependency monitoring

npm audit
    |
    v
Jenkins build-time dependency check
```

They are related, but they are not the same thing.

---

# 9. Step 7 – SonarQube Analysis

### Theory

Tests tell us whether the application behaves correctly.

SonarQube looks at the **quality and security of the source code**.

It can identify:

* Bugs
* Vulnerabilities
* Code smells
* Duplicated code
* Coverage information

### `sonar-project.properties`

```properties
sonar.projectKey=enterprise-pipeline
sonar.projectName=enterprise-pipeline

sonar.sources=src
sonar.tests=tests

sonar.javascript.lcov.reportPaths=coverage/lcov.info

sonar.exclusions=node_modules/**
```

The important connection is:

```text
Jest
  |
  v
coverage/lcov.info
  |
  v
SonarQube
```

So the coverage generated by Jest can be used by SonarQube.

---

# 10. Step 8 – SonarQube Quality Gate

### Theory

After SonarQube analyzes the project, Jenkins should not automatically continue.

It first asks:

> Did the project pass the SonarQube Quality Gate?

### Jenkins stage

```groovy
stage('Quality Gate') {
    steps {

        timeout(time: 5, unit: 'MINUTES') {

            waitForQualityGate abortPipeline: true
        }
    }
}
```

Flow:

```text
SonarQube Analysis
       |
       v
Quality Gate
       |
    +--+--+
    |     |
   PASS  FAIL
    |     |
    v     v
Continue Stop
```

`abortPipeline: true` means the Jenkins pipeline stops when the Quality Gate fails.

---

# 11. Step 9 – Docker Build

### Theory

Once the source code passes testing, security checks, and quality checks, we can package the application.

Docker creates a portable container image.

### `Dockerfile`

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install --omit=dev

COPY src ./src

EXPOSE 3000

CMD ["node", "src/app.js"]
```

Jenkins runs:

```bash
docker build -t enterprise-pipeline:1 .
```

The final `.` means:

> Use the current directory as the Docker build context.

Flow:

```text
Dockerfile
    +
Application Code
    |
    v
docker build
    |
    v
Docker Image
```

---

# 12. Step 10 – ECR Login

### Theory

The Docker image is currently on the Jenkins agent.

We need to authenticate Docker with **Amazon ECR** before pushing the image.

Jenkins executes:

```bash
aws ecr get-login-password \
--region us-east-1 \
| docker login \
--username AWS \
--password-stdin \
ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com
```

### Jenkins stage

```groovy
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
```

Flow:

```text
Jenkins Agent
      |
      v
AWS CLI
      |
      v
AWS ECR Authentication
      |
      v
Docker Login
```

---

# 13. Step 11 – Docker Tag

### Theory

Docker needs the ECR repository address in the image tag before it can push the image.

Local image:

```text
enterprise-pipeline:25
```

ECR image:

```text
ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/enterprise-pipeline:25
```

Jenkins executes:

```bash
docker tag \
enterprise-pipeline:25 \
ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/enterprise-pipeline:25
```

---

# 14. Step 12 – Push Image to ECR

### Theory

Now the image has:

1. Passed tests
2. Passed dependency checks
3. Passed SonarQube
4. Passed Quality Gate
5. Been built as a Docker image
6. Been tagged for ECR
7. Authenticated with ECR

Now Jenkins pushes it.

```bash
docker push \
ACCOUNT_ID.dkr.ecr.us-east-1.amazonaws.com/enterprise-pipeline:25
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

# 15. Complete Jenkinsfile

```groovy
pipeline {

    agent {
        label 'agent-1'
    }

    environment {

        AWS_ACCOUNT_ID = 'YOUR_AWS_ACCOUNT_ID'
        AWS_REGION     = 'us-east-1'
        ECR_REPO_NAME  = 'enterprise-pipeline'

        APP_NAME  = 'enterprise-pipeline'
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
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
        }

        stage('Dependency Security') {
            steps {
                sh 'npm audit --audit-level=high'
            }
        }

        stage('SonarQube Analysis') {
            steps {

                withSonarQubeEnv('sonarqube') {

                    sh '''
                        sonar-scanner \
                        -Dsonar.projectKey=enterprise-pipeline \
                        -Dsonar.sources=src \
                        -Dsonar.tests=tests \
                        -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
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

        stage('Docker Tag') {
            steps {

                sh '''
                    docker tag \
                    ${APP_NAME}:${IMAGE_TAG} \
                    ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO_NAME}:${IMAGE_TAG}
                '''
            }
        }

        stage('Push to ECR') {
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
    }
}
```

---

# 16. Credentials and Connections

The pipeline communicates with several systems.

```text
GitHub
   |
   | Git credentials
   v
Jenkins
   |
   | SonarQube token
   v
SonarQube

Jenkins
   |
   | AWS IAM credentials / role
   v
AWS ECR
```

### Important

Never put passwords or tokens directly into the Jenkinsfile.

Use:

```text
Jenkins Credentials
        +
AWS IAM
        +
SonarQube Credentials
        +
GitHub Credentials
```

---

# 17. Jenkins Agent Requirements

Because the pipeline uses:

```groovy
agent {
    label 'agent-1'
}
```

the tools must be available on `agent-1`.

Check:

```bash
java -version
git --version
node --version
npm --version
docker --version
aws --version
sonar-scanner --version
```

**Remember:** the commands execute on the Jenkins agent, not magically on the controller.

---

# 18. Final Flow to Remember

When you look at this:

```text
1. GitHub
      ↓
2. Checkout
      ↓
3. npm install
      ↓
4. Unit Test + Coverage
      ↓
5. npm audit
      ↓
6. SonarQube
      ↓
7. Quality Gate
      ↓
8. Docker Build
      ↓
9. ECR Login
      ↓
10. Docker Tag
      ↓
11. Docker Push
      ↓
12. AWS ECR
```

Think:

```text
GET CODE
   ↓
PREPARE
   ↓
TEST
   ↓
SECURITY
   ↓
CODE QUALITY
   ↓
APPROVAL
   ↓
PACKAGE
   ↓
STORE
```

That's the complete concept of this pipeline.
