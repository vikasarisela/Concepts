# Jenkins CI/CD Pipeline – Complete Notes

## 1. Pipeline Flow

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
   +--> Checkout
   |
   +--> npm install
   |
   +--> Unit Test + Coverage
   |
   +--> npm audit
   |
   +--> SonarQube Analysis
   |
   +--> Quality Gate
   |
   +--> Docker Build
   |
   +--> ECR Login
   |
   +--> Docker Push
   |
   v
AWS ECR
```

**Dependabot** runs separately in GitHub and checks for vulnerable/outdated dependencies.

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

# 3. Application Code

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

---

# 4. Unit Test

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

This is **Unit Testing**.

---

# 5. package.json

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

### Important

```bash
npm install
```

installs dependencies into:

```text
node_modules/
```

and creates/updates:

```text
package-lock.json
```

---

# 6. Dependabot

### `.github/dependabot.yml`

```yaml
version: 2

updates:
  - package-ecosystem: "npm"
    directory: "/"
    schedule:
      interval: "weekly"
```

### What it does

```text
GitHub
   |
   v
Dependabot
   |
   v
Checks npm dependencies
   |
   v
Vulnerability found
   |
   v
Alert / Pull Request
```

Dependabot is mainly for **dependency security and updates**.

---

# 7. SonarQube

### `sonar-project.properties`

```properties
sonar.projectKey=enterprise-pipeline
sonar.projectName=enterprise-pipeline

sonar.sources=src
sonar.tests=tests

sonar.javascript.lcov.reportPaths=coverage/lcov.info
sonar.exclusions=node_modules/**
```

SonarQube checks:

```text
Bugs
Vulnerabilities
Code Smells
Code Coverage
Duplications
```

---

# 8. Dockerfile

```dockerfile
FROM node:20-alpine

WORKDIR /app

COPY package*.json ./

RUN npm install --omit=dev

COPY src ./src

EXPOSE 3000

CMD ["node", "src/app.js"]
```

Docker packages the application into an image.

---

# 9. Jenkinsfile

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

# 10. Jenkins Stages in Simple Words

| Stage        | What it does                      |
| ------------ | --------------------------------- |
| Checkout     | Gets code from GitHub             |
| npm install  | Installs dependencies             |
| Unit Test    | Tests application                 |
| Coverage     | Measures tested code              |
| npm audit    | Checks dependency vulnerabilities |
| SonarQube    | Checks code quality/security      |
| Quality Gate | Allows or stops pipeline          |
| Docker Build | Creates Docker image              |
| ECR Login    | Authenticates to AWS ECR          |
| Docker Tag   | Gives ECR tag                     |
| Push to ECR  | Uploads image                     |

---

# 11. Credentials / Connections

```text
GitHub
   |
   | Git credentials
   v
Jenkins

Jenkins
   |
   | SonarQube token
   v
SonarQube

Jenkins
   |
   | AWS IAM credentials/role
   v
AWS ECR
```

### Never put secrets directly in Jenkinsfile.

Use:

```text
Jenkins Credentials
AWS IAM Role
GitHub Credentials
SonarQube Credentials
```

---

# 12. Jenkins Agent

Because we use:

```groovy
agent {
    label 'agent-1'
}
```

the required tools must be available on **agent-1**:

```bash
java -version
git --version
node --version
npm --version
docker --version
aws --version
sonar-scanner --version
```

---

# 13. First Pipeline Run

When the pipeline runs for the first time:

```text
GitHub
   |
   v
Checkout
   |
   v
npm install
   |
   v
node_modules created
   |
   v
npm test
   |
   v
coverage generated
   |
   v
SonarQube
   |
   v
Docker build
   |
   v
ECR push
```

---

# 14. When a Dependency Has a Vulnerability

```text
Dependabot
    |
    v
Finds vulnerable dependency
    |
    v
Update package.json
    |
    v
npm install
    |
    v
package-lock.json updated
    |
    v
git add .
git commit
git push
    |
    v
Jenkins runs again
```

---

# 15. Most Important Commands

### Git

```bash
git add .
git commit -m "Update application"
git push origin main
```

### npm

```bash
npm install
npm test
npm audit --audit-level=high
```

### Docker

```bash
docker build -t enterprise-pipeline:1 .
docker images
```

### AWS ECR

```bash
aws ecr get-login-password --region us-east-1
```

---

# 16. Final Understanding

Remember these four areas:

```text
DEPENDENCY SECURITY
        |
   Dependabot
        |
        v

CODE QUALITY
        |
    SonarQube
        |
        v

CI/CD
        |
     Jenkins
        |
        v

CONTAINER
        |
      Docker
        |
        v

IMAGE STORAGE
        |
      AWS ECR
```

The complete pipeline is therefore:

```text
GitHub
  ↓
Jenkins
  ↓
npm install
  ↓
Unit Test + Coverage
  ↓
npm audit
  ↓
SonarQube
  ↓
Quality Gate
  ↓
Docker Build
  ↓
AWS ECR
  ↓
Docker Push
```
