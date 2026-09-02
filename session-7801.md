Use this format
# End-to-End DevSecOps CI/CD Pipeline — Notes

These notes cover the complete Jenkins DevSecOps pipeline and the connections between GitHub, Dependabot, SonarQube, Docker, Trivy, AWS ECR, and Jenkins.

---

## 1. Overall Architecture

```text
Developer
    ↓
GitHub
    ↓
Jenkins
    ↓
Checkout Code
    ↓
Install Dependencies
    ↓
Dependabot Security Gate
    ↓
Unit Tests
    ↓
Generate Coverage
    ↓
SonarQube Scan
    ↓
SonarQube Quality Gate
    ↓
Docker Build
    ↓
Trivy Image Scan
    ↓
Push Image to AWS ECR
    ↓
Deploy

The purpose is:

Only tested, quality-checked, and security-checked code should be deployed.

2. GitHub

GitHub is where your application source code is stored.

Example:

enterprise-pipeline/
├── package.json
├── package-lock.json
├── src/
├── test/
├── Dockerfile
└── Jenkinsfile

The developer makes a change:

git add .
git commit -m "updated application"
git push origin main

This triggers Jenkins.

3. Jenkins

Jenkins is the orchestrator.

It executes all the stages:

GitHub
   ↓
Jenkins
   ↓
Tests
   ↓
Security
   ↓
Quality
   ↓
Docker
   ↓
ECR
   ↓
Deployment

Jenkins itself doesn't perform every check.

It calls the appropriate tools.

4. Install Dependencies

For Node.js:

stage('Install Dependencies') {
    steps {
        sh 'npm install'
    }
}

This reads:

package.json
      +
package-lock.json
      ↓
npm
      ↓
node_modules/
5. Dependabot Security Gate

Dependabot checks your project dependencies for known vulnerabilities.

Jenkins
   ↓
GitHub API
   ↓
Dependabot Alerts
   ↓
Open vulnerabilities?

If vulnerabilities are found:

Dependabot
    ↓
Open vulnerability
    ↓
❌ Jenkins aborts

If no vulnerabilities are found:

No open vulnerabilities
        ↓
Continue
6. Unit Testing

Jenkins runs the application's unit tests.

stage('Unit Test') {
    steps {
        sh 'npm test'
    }
}

If the test fails:

Unit Test
    ↓
FAIL
    ↓
❌ Jenkins stops
7. Generate Coverage

Run:

npm test -- --coverage

This generates:

coverage/
└── lcov.info

The lcov.info file contains information about which executable lines, functions, and branches were exercised by the tests.

8. SonarQube Scan

Jenkins starts SonarScanner:

stage('SonarQube Scan') {
    steps {
        withSonarQubeEnv('SonarQube') {
            sh '''
                sonar-scanner \
                  -Dsonar.projectKey=enterprise-pipeline \
                  -Dsonar.sources=. \
                  -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info
            '''
        }
    }
}

Flow:

Jenkins
   ↓
SonarScanner
   ↓
Source Code
   +
coverage/lcov.info
   ↓
SonarQube Server
9. SonarQube Quality Gate

Example:

Quality Gate
────────────────────────
Coverage          >= 80%
New Bugs          = 0
Vulnerabilities   = 0
Duplications      <= 3%
────────────────────────
       PASS / FAIL

If all conditions pass:

Quality Gate = PASS

If any condition fails:

Quality Gate = FAIL
        ↓
❌ Jenkins stops
10. How Jenkins Gets the Result

SonarQube sends the Quality Gate result back to Jenkins through a webhook.

Jenkins
   ↓
SonarScanner
   ↓
SonarQube
   ↓
Quality Gate
   ↓
PASS / FAIL
   ↓
Webhook
   ↓
Jenkins

Jenkins waits using:

waitForQualityGate abortPipeline: true
11. Docker Build

After the quality gates pass:

docker build -t enterprise-pipeline:latest .

Flow:

Application
    +
Dockerfile
    ↓
Docker Build
    ↓
Docker Image
12. Trivy Image Scan

Trivy scans the Docker image for vulnerabilities.

Docker Image
      ↓
    Trivy
      ↓
Vulnerability Scan

If unacceptable vulnerabilities are found:

HIGH / CRITICAL
       ↓
❌ STOP

Otherwise:

PASS
 ↓
Continue
13. Push to AWS ECR

After all security and quality checks pass:

Docker Image
     ↓
AWS ECR

Example:

193849563622.dkr.ecr.us-east-1.amazonaws.com/
enterprise-pipeline:latest
14. Complete Pipeline
Developer
    ↓
GitHub
    ↓
Jenkins
    ↓
Checkout
    ↓
npm install
    ↓
Dependabot Security Gate
    ↓
Unit Tests
    ↓
Coverage
    ↓
SonarQube Scan
    ↓
SonarQube Quality Gate
    ↓
Docker Build
    ↓
Trivy Image Scan
    ↓
Push to ECR
    ↓
Deploy
15. Final Principle

Test → Analyze → Secure → Build → Scan → Push → Deploy

This is a DevSecOps CI/CD Pipeline.


### Important

Make sure your file is saved as:

```text
README.md

or:

DevSecOps-Notes.md

and do not put the entire document inside one code block.

For example, this is wrong:

```markdown
# My Notes

## Section 1
...
```

The outer code fence makes GitHub display the Markdown as plain text.

You want the actual file to contain:

# My Notes

## Section 1

Some text.

```text
Jenkins
   ↓
SonarQube