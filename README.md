
# 1. Create a Node.js project with Jest
mkdir cicd-nodejs
cd cicd-nodejs
npm init -y
npm install --save-dev jest
npm pkg set scripts.test="jest"

# Create source code
mkdir src
echo "function add(a, b) { return a + b; } module.exports = add;" > src/index.js

# Create test file
mkdir tests
echo "const add = require('../src/index'); test('adds 2 + 3', () => { expect(add(2, 3)).toBe(5); });" > tests/index.test.js

# Run test locally
npm test

# 2. Initialize Git and push project to GitHub
git init
git add .
git commit -m "Initial Node.js project"
git branch -M main

# Create an empty repository named cicd-nodejs on GitHub first
git remote add origin https://github.com/YOUR_USERNAME/cicd-nodejs.git
git push -u origin main

# 3. Create GitHub Actions workflow directory and file
mkdir -p .github/workflows

# Create the workflow file (Windows Git Bash)
cat > .github/workflows/ci.yml <<'EOF'
name: Node.js CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout source code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: npm

      - name: Install dependencies
        run: npm install

      - name: Run test cases
        run: npm test
EOF

# 4. Commit and run GitHub Actions workflow
git add .
git commit -m "Add GitHub Actions CI workflow"
git push

# Verify GitHub Actions:
# GitHub repository > Actions > Node.js CI > workflow run

# 5. Install and configure Jenkins
# Install Java and Jenkins using the official instructions for your OS:
# https://www.jenkins.io/doc/book/installing/
# Start Jenkins, open http://localhost:8080,
# unlock Jenkins, install suggested plugins, and create an admin user.
# Ensure Git, Node.js, and npm are available to the Jenkins service.

# 6. Create a Jenkins Pipeline job
# Jenkins dashboard > New Item > cicd-nodejs-pipeline
# Select Pipeline > OK

# 7. Create Jenkinsfile in the project root
cat > Jenkinsfile <<'EOF'
pipeline {
    agent any

    tools {
        nodejs 'NodeJS'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install') {
            steps {
                sh 'npm install'
            }
        }

        stage('Test') {
            steps {
                sh 'npm test'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build --if-present'
            }
        }
    }
}
EOF

# Configure Jenkins NodeJS tool:
# Manage Jenkins > Tools > NodeJS installations
# Add installation named NodeJS and select a Node.js version.
# In the Pipeline job, select "Pipeline script from SCM",
# choose Git, enter your repository URL, and set Script Path to Jenkinsfile.

# Push Jenkinsfile to GitHub
git add Jenkinsfile
git commit -m "Add Jenkins pipeline"
git push

# 8. Execute and verify both pipelines
# GitHub: Repository > Actions > open the latest run.
# Jenkins: Open the pipeline job > Build Now > Console Output.
# Confirm Checkout, Install, Test, and Build stages succeed.

# 9. Capture screenshots
# Screenshot 1: GitHub repository > Actions > successful workflow run.
# Screenshot 2: Jenkins pipeline > Console Output showing success.

# 10. Compare GitHub Actions and Jenkins
# GitHub Actions: integrated with GitHub; workflows use YAML.
# Jenkins: separately installed automation server; pipeline uses Jenkinsfile.
# Both can automate checkout, dependency installation, testing, and builds.
# GitHub Actions uses hosted runners or self-hosted runners.
# Jenkins uses configured agents/nodes.

# 11. Conclusion
# Both GitHub Actions and Jenkins successfully automate CI/CD.
# GitHub Actions provides convenient GitHub integration, while Jenkins
# offers flexible pipeline configuration and extensibility.
