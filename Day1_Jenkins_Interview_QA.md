# 🚀 DevOps Interview Preparation — Day 1: Jenkins
### For 3 Years Experience | 100+ Questions & Answers

> **Study Plan:** Jenkins → AWS → Docker → Kubernetes → Terraform → Linux → Git → Ansible → Monitoring
> Use `Ctrl+Shift+V` in VS Code to preview this file beautifully.

---

## 📋 Table of Contents
1. [Basic Concepts (Q1–Q20)](#1-basic-concepts)
2. [Pipeline & Jenkinsfile (Q21–Q45)](#2-pipeline--jenkinsfile)
3. [Plugins & Integrations (Q46–Q60)](#3-plugins--integrations)
4. [Security & Best Practices (Q61–Q75)](#4-security--best-practices)
5. [Troubleshooting & Scenarios (Q76–Q90)](#5-troubleshooting--scenarios)
6. [Advanced & Real-World (Q91–Q110)](#6-advanced--real-world)

---

## 1. Basic Concepts

**Q1. What is Jenkins?**
> Jenkins is an open-source automation server written in Java. It helps automate building, testing, and deploying software — this is called CI/CD (Continuous Integration / Continuous Deployment).

---

**Q2. What is CI/CD? Explain simply.**
> - **CI (Continuous Integration):** Developers push code frequently. Jenkins automatically builds and tests it.
> - **CD (Continuous Delivery):** After tests pass, the code is automatically prepared for deployment.
> - **CD (Continuous Deployment):** Code goes live to production automatically without manual approval.

---

**Q3. What is a Jenkins Job / Project?**
> A Job is a task you configure in Jenkins. Example: "When someone pushes code to GitHub, build the project and run tests." Types: Freestyle, Pipeline, Multibranch Pipeline, Folder.

---

**Q4. What is a Jenkins Pipeline?**
> A Pipeline is a set of automated steps defined in code (called Jenkinsfile) that tells Jenkins what to do — build, test, deploy, notify, etc.

---

**Q5. What is a Jenkinsfile?**
> A Jenkinsfile is a text file (written in Groovy DSL) that lives in your code repository and defines the entire CI/CD pipeline. It's like a recipe for Jenkins.

---

**Q6. What are the two types of Jenkins Pipeline syntax?**
> - **Declarative Pipeline:** Simpler, structured format. Recommended for beginners.
> - **Scripted Pipeline:** More powerful, uses full Groovy. For complex logic.

---

**Q7. What is a Jenkins Agent / Node?**
> The **Master (Controller)** manages jobs and scheduling. An **Agent (Node/Slave)** is a machine that actually runs the build. You can have many agents to run jobs in parallel.

---

**Q8. What is an Executor in Jenkins?**
> An Executor is a slot on an agent that runs one build at a time. If an agent has 3 executors, it can run 3 builds simultaneously.

---

**Q9. What is a Jenkins Build?**
> A Build is one run/execution of a Jenkins job. Every time a job runs, it creates a new build with a build number (#1, #2, #3...).

---

**Q10. What is a Jenkins Workspace?**
> A Workspace is the directory on the agent machine where Jenkins checks out your code and runs the build commands.

---

**Q11. What is the difference between Freestyle and Pipeline jobs?**
> | Feature | Freestyle | Pipeline |
> |---|---|---|
> | Configuration | GUI only | Code (Jenkinsfile) |
> | Version control | No | Yes |
> | Complex flows | Limited | Fully supported |
> | Recommended | Simple tasks | All CI/CD workflows |

---

**Q12. What is Blue Ocean in Jenkins?**
> Blue Ocean is a modern UI plugin for Jenkins that makes pipelines easier to visualize. It shows a graphical view of pipeline stages and steps.

---

**Q13. What is a Jenkins Plugin?**
> Plugins extend Jenkins functionality. Examples: Git plugin (for GitHub), Docker plugin, Kubernetes plugin, Slack plugin. Jenkins has 1800+ plugins.

---

**Q14. What is a Trigger in Jenkins?**
> A Trigger decides when a Jenkins job should start automatically. Types:
> - **SCM Polling:** Jenkins checks GitHub every X minutes
> - **Webhook:** GitHub notifies Jenkins on every push (better!)
> - **Scheduled (Cron):** Run at specific times
> - **Upstream trigger:** Run after another job finishes

---

**Q15. What is a Webhook in Jenkins context?**
> A Webhook is a URL that GitHub/GitLab calls when someone pushes code. Jenkins listens at this URL and starts a build immediately. Faster than polling.

---

**Q16. What is SCM in Jenkins?**
> SCM stands for Source Code Management. Jenkins uses it to connect to repositories like GitHub, GitLab, or Bitbucket to pull your code.

---

**Q17. What are Build Parameters in Jenkins?**
> Parameters let you pass values to a job at runtime. Example: Pass `ENVIRONMENT=staging` or `VERSION=1.2.3` when triggering a job. Types: String, Boolean, Choice, File.

---

**Q18. What is a Multibranch Pipeline?**
> Jenkins automatically creates pipelines for every branch in your repository. When you create a new branch, Jenkins detects it and creates a pipeline for it using the Jenkinsfile in that branch.

---

**Q19. What is Jenkins Master-Slave architecture?**
> - **Master:** Schedules jobs, stores configuration, serves the UI
> - **Slaves/Agents:** Run the actual builds
> - This distributes load and allows running builds on different OS/environments

---

**Q20. How does Jenkins connect to agents?**
> Two ways:
> - **SSH:** Jenkins connects to the agent via SSH (Linux agents)
> - **JNLP/WebSocket:** Agent connects back to master (useful when agent is behind firewall)

---

## 2. Pipeline & Jenkinsfile

**Q21. Write a simple Declarative Pipeline example.**
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                echo 'Building the application...'
                sh 'mvn clean package'
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests...'
                sh 'mvn test'
            }
        }
        stage('Deploy') {
            steps {
                echo 'Deploying...'
                sh './deploy.sh'
            }
        }
    }
}
```

---

**Q22. What are the main sections of a Declarative Pipeline?**
> - `pipeline` — root block
> - `agent` — where to run
> - `stages` — list of stages
> - `stage` — single phase (Build, Test, Deploy)
> - `steps` — actual commands inside a stage
> - `post` — actions after pipeline (success, failure, always)

---

**Q23. What is the `agent` directive?**
> It tells Jenkins where to run the pipeline:
> - `agent any` — run on any available agent
> - `agent none` — no global agent, define per stage
> - `agent { label 'linux' }` — run on agent with label 'linux'
> - `agent { docker 'node:18' }` — run inside Docker container

---

**Q24. What is the `post` section in Jenkinsfile?**
```groovy
post {
    always {
        echo 'This runs always'
        cleanWs() // clean workspace
    }
    success {
        echo 'Build passed!'
        slackSend message: '✅ Build successful'
    }
    failure {
        echo 'Build failed!'
        mail to: 'team@company.com', subject: 'Build Failed'
    }
}
```

---

**Q25. What is `environment` block in Jenkinsfile?**
```groovy
pipeline {
    environment {
        APP_NAME = 'myapp'
        DOCKER_IMAGE = "myrepo/${APP_NAME}:${BUILD_NUMBER}"
    }
    ...
}
```
> Used to define environment variables available throughout the pipeline.

---

**Q26. What are Jenkins Environment Variables?**
> Built-in variables Jenkins provides automatically:
> - `BUILD_NUMBER` — current build number
> - `JOB_NAME` — name of the job
> - `BUILD_URL` — URL of the build
> - `GIT_BRANCH` — current git branch
> - `WORKSPACE` — path to workspace directory

---

**Q27. How do you use credentials in a Jenkinsfile?**
```groovy
withCredentials([usernamePassword(
    credentialsId: 'docker-hub-creds',
    usernameVariable: 'DOCKER_USER',
    passwordVariable: 'DOCKER_PASS'
)]) {
    sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
}
```

---

**Q28. What is parallel execution in Jenkins Pipeline?**
```groovy
stage('Test in Parallel') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'mvn test -Dtest=UnitTests' }
        }
        stage('Integration Tests') {
            steps { sh 'mvn test -Dtest=IntegrationTests' }
        }
    }
}
```
> Runs multiple stages at the same time, reducing total pipeline time.

---

**Q29. What is `when` directive in Pipeline?**
```groovy
stage('Deploy to Prod') {
    when {
        branch 'main'  // Only run this stage on main branch
    }
    steps {
        sh './deploy-prod.sh'
    }
}
```

---

**Q30. How do you handle timeouts in Jenkins?**
```groovy
options {
    timeout(time: 30, unit: 'MINUTES')
}
// or inside a step:
timeout(time: 5, unit: 'MINUTES') {
    sh './long-running-script.sh'
}
```

---

**Q31. What is `stash` and `unstash` in Jenkins?**
> Used to pass files between stages that run on different agents:
```groovy
stage('Build') {
    steps {
        sh 'mvn package'
        stash name: 'build-artifacts', includes: 'target/*.jar'
    }
}
stage('Deploy') {
    agent { label 'deploy-server' }
    steps {
        unstash 'build-artifacts'
        sh 'deploy target/*.jar'
    }
}
```

---

**Q32. What is `input` step in Jenkins?**
```groovy
stage('Approve Production Deploy') {
    steps {
        input message: 'Deploy to production?', ok: 'Yes, Deploy!'
    }
}
```
> Pauses the pipeline and waits for manual human approval before continuing.

---

**Q33. What is `retry` in Jenkins Pipeline?**
```groovy
steps {
    retry(3) {
        sh './flaky-test.sh'  // Retries up to 3 times if it fails
    }
}
```

---

**Q34. What is `catchError` in Jenkins?**
```groovy
steps {
    catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
        sh './test.sh'  // If this fails, stage is marked failed but build continues
    }
}
```

---

**Q35. How do you archive artifacts in Jenkins?**
```groovy
post {
    always {
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        junit 'target/surefire-reports/*.xml'  // publish test results
    }
}
```

---

**Q36. What is a Shared Library in Jenkins?**
> A Shared Library is reusable Groovy code stored in a separate Git repo. Multiple pipelines can use it to avoid code duplication.
```groovy
// In Jenkinsfile
@Library('my-shared-library') _
myDeployFunction(env: 'staging')
```

---

**Q37. What is `sh` vs `bat` in Jenkins?**
> - `sh` — runs shell commands on Linux/Mac agents
> - `bat` — runs batch commands on Windows agents

---

**Q38. How do you read a file in Jenkins Pipeline?**
```groovy
def content = readFile('config/app.properties')
echo content
```

---

**Q39. What is `currentBuild` in Jenkins?**
> `currentBuild` is an object with info about the current running build:
> - `currentBuild.result` — SUCCESS, FAILURE, UNSTABLE
> - `currentBuild.displayName` — build display name
> - `currentBuild.description` — set a description
> - `currentBuild.duration` — how long it ran

---

**Q40. How do you trigger another job from a Jenkinsfile?**
```groovy
build job: 'downstream-job', 
      parameters: [string(name: 'ENV', value: 'staging')],
      wait: true  // wait for it to finish
```

---

**Q41. What is `withEnv` in Jenkins?**
```groovy
withEnv(['PATH+EXTRA=/usr/local/bin', 'DEBUG=true']) {
    sh 'my-command'
}
```
> Temporarily sets environment variables for a block of steps.

---

**Q42. How do you checkout code in a Pipeline?**
```groovy
checkout scm  // checks out the repo that triggered the build
// or explicitly:
checkout([$class: 'GitSCM',
    branches: [[name: '*/main']],
    userRemoteConfigs: [[url: 'https://github.com/org/repo.git']]])
```

---

**Q43. What is `node` in Scripted Pipeline?**
```groovy
node('linux-agent') {
    stage('Build') {
        sh 'make build'
    }
}
```
> In Scripted Pipeline, `node` allocates an executor on an agent (like `agent` in Declarative).

---

**Q44. How do you set build status/description dynamically?**
```groovy
currentBuild.displayName = "#${BUILD_NUMBER} - ${GIT_BRANCH}"
currentBuild.description = "Deployed version ${VERSION} to ${ENV}"
```

---

**Q45. What is `options` block in Declarative Pipeline?**
```groovy
options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
    timestamps()
    timeout(time: 1, unit: 'HOURS')
}
```

---

## 3. Plugins & Integrations

**Q46. Name 10 important Jenkins plugins.**
> 1. **Git Plugin** — GitHub/GitLab integration
> 2. **Pipeline Plugin** — enables Jenkinsfile pipelines
> 3. **Blue Ocean** — modern UI
> 4. **Docker Pipeline** — run builds in Docker
> 5. **Kubernetes Plugin** — run agents as K8s pods
> 6. **Credentials Plugin** — store secrets securely
> 7. **Slack Notification** — send Slack alerts
> 8. **Email Extension** — send detailed emails
> 9. **SonarQube Scanner** — code quality
> 10. **JUnit Plugin** — publish test reports

---

**Q47. How do you integrate Jenkins with GitHub?**
> 1. Install GitHub plugin in Jenkins
> 2. Create a Personal Access Token (PAT) on GitHub
> 3. Add the PAT as Jenkins credentials
> 4. Configure job with GitHub repo URL
> 5. Set up Webhook on GitHub → point to `http://jenkins-url/github-webhook/`

---

**Q48. How do you integrate Jenkins with Docker?**
```groovy
pipeline {
    agent {
        docker {
            image 'maven:3.9-openjdk-17'
            args '-v /root/.m2:/root/.m2'  // cache Maven repo
        }
    }
    stages {
        stage('Build') {
            steps { sh 'mvn package' }
        }
    }
}
```

---

**Q49. How do you integrate Jenkins with SonarQube?**
```groovy
stage('Code Quality') {
    steps {
        withSonarQubeEnv('SonarQube-Server') {
            sh 'mvn sonar:sonar'
        }
    }
}
stage('Quality Gate') {
    steps {
        waitForQualityGate abortPipeline: true
    }
}
```

---

**Q50. How do you send Slack notifications from Jenkins?**
```groovy
post {
    success {
        slackSend(
            channel: '#deployments',
            color: 'good',
            message: "✅ ${JOB_NAME} #${BUILD_NUMBER} passed!"
        )
    }
    failure {
        slackSend(
            channel: '#deployments',
            color: 'danger',
            message: "❌ ${JOB_NAME} #${BUILD_NUMBER} FAILED! ${BUILD_URL}"
        )
    }
}
```

---

**Q51. How does Jenkins integrate with Kubernetes?**
> Using the **Kubernetes Plugin**, Jenkins creates a pod in K8s for each build (ephemeral agents), runs the build, then deletes the pod. This is very scalable.
```groovy
agent {
    kubernetes {
        yaml '''
        spec:
          containers:
          - name: maven
            image: maven:3.9
        '''
    }
}
```

---

**Q52. How do you publish test reports in Jenkins?**
```groovy
post {
    always {
        junit '**/target/surefire-reports/*.xml'
        publishHTML([reportDir: 'target/site', reportFiles: 'index.html', reportName: 'Test Report'])
    }
}
```

---

**Q53. What is the Credentials Binding plugin?**
> It allows you to use stored Jenkins credentials (passwords, SSH keys, tokens) as environment variables in your pipeline without exposing them in logs.

---

**Q54. How do you use SSH credentials in Jenkins?**
```groovy
withCredentials([sshUserPrivateKey(
    credentialsId: 'deploy-server-key',
    keyFileVariable: 'SSH_KEY'
)]) {
    sh 'ssh -i $SSH_KEY user@server ./deploy.sh'
}
```

---

**Q55. What is the Role-Based Authorization Strategy plugin?**
> It lets you control who can do what in Jenkins. You can create roles like:
> - `admin` — full access
> - `developer` — can view and trigger builds
> - `viewer` — read-only access
> Then assign these roles to users or groups.

---

**Q56. What is the Job DSL plugin?**
> Lets you create Jenkins jobs programmatically using Groovy code instead of the UI. Useful for managing many similar jobs as code.

---

**Q57. How does Jenkins integrate with Nexus/Artifactory?**
> After building an artifact (JAR/WAR/Docker image), Jenkins uploads it to Nexus or Artifactory as a version. Later stages or other pipelines can download it from there.

---

**Q58. What is the OWASP Dependency Check plugin?**
> It scans your project dependencies for known security vulnerabilities (CVEs) and reports them in Jenkins. Important for security compliance.

---

**Q59. How do you integrate Jenkins with Ansible?**
```groovy
stage('Deploy with Ansible') {
    steps {
        ansiblePlaybook(
            playbook: 'deploy.yml',
            inventory: 'inventory/production',
            credentialsId: 'ansible-ssh-key'
        )
    }
}
```

---

**Q60. What is the Parameterized Trigger plugin?**
> Allows one Jenkins job to trigger another job and pass parameters to it. Useful for chaining jobs in complex workflows.

---

## 4. Security & Best Practices

**Q61. How do you store secrets in Jenkins?**
> Use Jenkins **Credentials Store**:
> - Go to Manage Jenkins → Credentials
> - Add credentials (username/password, secret text, SSH key, certificate)
> - Reference them in pipeline with `credentials()` or `withCredentials()`
> - **Never** hardcode secrets in Jenkinsfile!

---

**Q62. What is Jenkins security realm?**
> Security realm controls how users **authenticate** (log in) to Jenkins:
> - **Jenkins own user database** — manages users internally
> - **LDAP** — integrate with company Active Directory
> - **GitHub OAuth** — login with GitHub account
> - **SAML** — enterprise SSO

---

**Q63. What is the difference between authentication and authorization in Jenkins?**
> - **Authentication:** Verifying WHO you are (login)
> - **Authorization:** Deciding WHAT you can do (permissions)
> Jenkins supports Matrix-based security for fine-grained authorization.

---

**Q64. What is CSRF protection in Jenkins?**
> CSRF (Cross-Site Request Forgery) protection prevents attackers from tricking users into making unauthorized requests to Jenkins. Always keep it enabled. Requires a crumb token for API calls.

---

**Q65. How do you secure Jenkins API?**
> - Use API tokens instead of passwords
> - Create token: User Profile → API Token → Add New Token
> - Use in scripts: `curl -u username:api-token https://jenkins/job/myjob/build`

---

**Q66. What are best practices for Jenkins security?**
> 1. Run Jenkins on HTTPS
> 2. Keep Jenkins and plugins updated
> 3. Use the Principle of Least Privilege (minimal permissions)
> 4. Store secrets in Credentials, never in code
> 5. Enable CSRF protection
> 6. Restrict script approvals (Script Security Plugin)
> 7. Use audit trail plugin to log actions
> 8. Disable unused plugins

---

**Q67. What is the Script Security plugin?**
> It controls which Groovy scripts can run in Jenkins pipelines. Scripts go through approval by an admin. This prevents malicious code from running.

---

**Q68. How do you handle Jenkins agent security?**
> - Connect agents via SSH with key-based auth (no passwords)
> - Run agent processes with dedicated low-privilege user
> - Don't run agent on the master node in production
> - Use ephemeral agents (Docker/K8s) — they're destroyed after each build

---

**Q69. What are Jenkins best practices for pipelines?**
> 1. Store Jenkinsfile in source code repo (version controlled)
> 2. Keep stages small and focused
> 3. Use Shared Libraries for common logic
> 4. Always clean up workspace (`cleanWs()`)
> 5. Use `timeout` to prevent hung builds
> 6. Send notifications on failure
> 7. Use declarative pipeline over scripted when possible
> 8. Limit `archiveArtifacts` — use Nexus/Artifactory instead

---

**Q70. What is `disableConcurrentBuilds()` and when to use it?**
> Prevents multiple builds of the same job from running simultaneously. Use when:
> - Deploying to an environment (avoid two deploys at once)
> - Running database migrations
> - Any operation that's not idempotent

---

**Q71. What is log rotation in Jenkins?**
> Jenkins stores build logs on disk. Without log rotation, this fills up disk space. Configure it with:
```groovy
options {
    buildDiscarder(logRotator(
        numToKeepStr: '30',       // keep last 30 builds
        artifactNumToKeepStr: '5' // but only keep artifacts for last 5
    ))
}
```

---

**Q72. How do you back up Jenkins?**
> Jenkins stores everything in `$JENKINS_HOME` directory. Backup strategies:
> - **ThinBackup plugin** — automated scheduled backups
> - **Periodic backup plugin**
> - Store Jenkins config as code (JCasC plugin) in Git
> - Snapshot the Jenkins server/EBS volume

---

**Q73. What is Jenkins Configuration as Code (JCasC)?**
> JCasC plugin lets you define entire Jenkins configuration in a YAML file, stored in Git. This makes Jenkins setup reproducible and versionable.
```yaml
jenkins:
  systemMessage: "Welcome to our CI/CD Server"
  numExecutors: 5
  securityRealm:
    local:
      allowsSignup: false
```

---

**Q74. What is the Audit Trail plugin?**
> It records all actions in Jenkins — who triggered a build, who changed a job, who deleted something. Essential for compliance and debugging.

---

**Q75. What is the principle of "Pipeline as Code"?**
> Pipeline as Code means your CI/CD pipeline is defined in a file (Jenkinsfile) that lives in your application's source code repository. Benefits:
> - Version controlled (track changes)
> - Code reviewed (PRs for pipeline changes)
> - Reproducible (same pipeline in every environment)
> - Auditable (who changed what, when)

---

## 5. Troubleshooting & Scenarios

**Q76. A build is stuck — how do you investigate?**
> 1. Check Console Output — look for the last line where it got stuck
> 2. Common causes: waiting for `input`, SSH connection hanging, test stuck in infinite loop
> 3. Check if agents are online: Manage Jenkins → Nodes
> 4. Look at thread dump: Manage Jenkins → System Information
> 5. If needed, abort the build and investigate the script

---

**Q77. Jenkins build fails with "No space left on device" — what do you do?**
> 1. Immediate: `docker system prune` on agent, remove old workspaces
> 2. Short-term: Configure log rotation, archive fewer artifacts
> 3. Long-term: Add more disk space, use external artifact storage (Nexus), use ephemeral agents

---

**Q78. How do you debug a Jenkinsfile?**
> 1. Use `echo` statements to print variable values
> 2. Use `sh 'env | sort'` to see all environment variables
> 3. Use Jenkins **Replay** feature — edit and re-run the pipeline without committing
> 4. Use `println` in Groovy sections
> 5. Check Pipeline Syntax generator in Jenkins UI for correct syntax

---

**Q79. What causes "Workspace is locked" error?**
> Another build is already using that workspace. Solutions:
> - Wait for the running build to finish
> - Use `disableConcurrentBuilds()` to prevent this
> - Or configure Jenkins to use a different workspace per build

---

**Q80. Build works locally but fails in Jenkins — common causes?**
> 1. **Different JDK/Node version** — Jenkins agent has different version
> 2. **Missing environment variables** — not set on Jenkins agent
> 3. **Missing tools** — tool not installed on the agent
> 4. **File permissions** — script not executable
> 5. **Network issues** — agent can't reach the internet or internal services
> 6. **Different OS** — dev on Mac, Jenkins on Linux

---

**Q81. How do you re-run only a failed stage (not full pipeline)?**
> Use **Stage Restart** feature in Blue Ocean. Or use `when { expression { return currentBuild.result != 'SUCCESS' } }` logic. Or use checkpointing with the `checkpoint` step.

---

**Q82. Jenkins master is slow — how do you troubleshoot?**
> 1. Check number of concurrent builds (too many?)
> 2. Check disk I/O on master — move builds to agents
> 3. Check memory: Manage Jenkins → Java Heap usage
> 4. Don't run builds on master — use agents only
> 5. Clean up old jobs, builds, and plugins
> 6. Upgrade Jenkins and JVM if outdated

---

**Q83. How do you handle flaky tests in Jenkins?**
> 1. Use `retry(3)` to retry flaky steps
> 2. Mark as UNSTABLE instead of FAILURE using `catchError`
> 3. Use **Test Retry Plugin** for automatic test retries
> 4. Investigate and fix root cause (shared state, timing issues)

---

**Q84. A Jenkins agent goes offline frequently — what do you check?**
> 1. Check agent machine resources (CPU, memory, disk)
> 2. Check SSH connectivity from master to agent
> 3. Check if agent Java process is crashing
> 4. Review agent logs: Manage Jenkins → Nodes → Agent → Log
> 5. Increase agent timeout settings
> 6. Consider switching to JNLP agent instead of SSH

---

**Q85. How do you roll back a failed deployment using Jenkins?**
> 1. **Blue-Green Strategy:** Switch load balancer back to previous environment
> 2. **Versioned Artifacts:** Re-deploy previous artifact version from Nexus
> 3. **Git Revert:** Run the pipeline on the previous commit
> 4. Best practice: Have a dedicated "Rollback" Jenkins job with `VERSION` parameter

---

**Q86. Jenkins throws "java.lang.OutOfMemoryError" — what do you do?**
> Increase Jenkins heap size. In Jenkins startup:
```bash
export JAVA_OPTS="-Xmx4g -Xms2g"
# or in /etc/default/jenkins:
JAVA_ARGS="-Xmx4096m"
```

---

**Q87. How do you handle long-running pipelines?**
> 1. Use `timeout` to kill if too long
> 2. Break into multiple stages with `input` for approval
> 3. Run independent stages in `parallel`
> 4. Use `stash/unstash` to pass results between fast stages
> 5. Consider splitting into multiple jobs

---

**Q88. What happens when Jenkins restarts during a build?**
> By default, the build is marked as failed. To survive restarts:
> - Use **Durable Pipeline Steps** (`options { durabilityHint('PERFORMANCE_OPTIMIZED') }`)
> - The Pipeline plugin stores build state to disk
> - Add `options { durabilityHint('MAX_SURVIVABILITY') }` for critical pipelines

---

**Q89. How do you prevent secrets from appearing in Jenkins console logs?**
> 1. Use `withCredentials` — Jenkins automatically masks the secret value in logs
> 2. Set `sh(script: '...', returnStdout: false)` for sensitive commands
> 3. Use the **Mask Passwords Plugin**
> 4. Never `echo` or `print` a credential value

---

**Q90. Pipeline worked before but suddenly fails with "method not allowed" — why?**
> Likely a Groovy method that needs to be approved via **Script Security Plugin**. Go to:
> Manage Jenkins → In-process Script Approval → Approve the pending method.

---

## 6. Advanced & Real-World

**Q91. How do you implement GitFlow with Jenkins Multibranch Pipeline?**
> ```
> feature/* branches → run unit tests only
> develop branch → run full tests + deploy to dev
> release/* branches → deploy to staging
> main branch → deploy to production (with approval)
> ```
> Each branch gets its own pipeline using the Jenkinsfile in that branch.

---

**Q92. How do you build and push a Docker image in Jenkins?**
```groovy
stage('Build Docker Image') {
    steps {
        script {
            def image = docker.build("myapp:${BUILD_NUMBER}")
            docker.withRegistry('https://registry.hub.docker.com', 'dockerhub-credentials') {
                image.push()
                image.push('latest')
            }
        }
    }
}
```

---

**Q93. How do you deploy to Kubernetes from Jenkins?**
```groovy
stage('Deploy to K8s') {
    steps {
        withKubeConfig([credentialsId: 'k8s-config']) {
            sh """
                kubectl set image deployment/myapp \
                    myapp=myrepo/myapp:${BUILD_NUMBER}
                kubectl rollout status deployment/myapp
            """
        }
    }
}
```

---

**Q94. What is a Declarative Pipeline matrix strategy?**
```groovy
matrix {
    axes {
        axis {
            name 'PLATFORM'
            values 'linux', 'mac', 'windows'
        }
        axis {
            name 'JAVA_VERSION'
            values '11', '17'
        }
    }
    stages {
        stage('Test') {
            steps { sh 'mvn test' }
        }
    }
}
```
> Automatically tests across all combinations (6 in this case).

---

**Q95. How do you implement a canary deployment with Jenkins?**
> 1. Deploy new version to 10% of pods: `kubectl scale deployment myapp-canary --replicas=1`
> 2. Monitor metrics (error rate, latency) for X minutes — use `input` for approval
> 3. If good: scale canary to 100%, scale old version to 0
> 4. If bad: scale canary to 0, rollback

---

**Q96. What is the difference between `sh` returnStatus and returnStdout?**
```groovy
// returnStatus: get exit code (0=success, non-zero=failure)
def status = sh(script: 'ls /tmp/file', returnStatus: true)
if (status != 0) { echo "File not found" }

// returnStdout: capture command output as string
def version = sh(script: 'cat version.txt', returnStdout: true).trim()
echo "Version: ${version}"
```

---

**Q97. How do you use Jenkins with Terraform?**
```groovy
stage('Terraform Apply') {
    steps {
        withCredentials([string(credentialsId: 'aws-secret-key', variable: 'AWS_SECRET_KEY')]) {
            dir('terraform/') {
                sh '''
                    terraform init
                    terraform plan -out=tfplan
                    terraform apply -auto-approve tfplan
                '''
            }
        }
    }
}
```

---

**Q98. What is Jenkins X?**
> Jenkins X is a cloud-native CI/CD platform built for Kubernetes. It's different from Jenkins:
> - Designed specifically for microservices on K8s
> - Automatically creates pipelines for new repositories
> - Built on Tekton (K8s-native pipeline engine)
> - More opinionated and automated than traditional Jenkins

---

**Q99. How do you scale Jenkins horizontally?**
> 1. **Multiple Agents:** Add more build agents as demand grows
> 2. **K8s Plugin:** Agents auto-scale as pods — spin up when needed, die after use
> 3. **EC2 Plugin:** Automatically provision EC2 instances as agents when load increases
> 4. **Horizontal Controller:** Multiple Jenkins controllers (advanced use case)

---

**Q100. How do you migrate jobs from Jenkins to GitLab CI or GitHub Actions?**
> 1. Export job configuration as XML: `http://jenkins/job/myjob/config.xml`
> 2. Review Jenkinsfile stages and translate to `.gitlab-ci.yml` or `.github/workflows/*.yml`
> 3. Move credentials to GitLab/GitHub secrets
> 4. Run both in parallel initially to validate
> 5. Cut over when confident

---

**Q101. What is the difference between Jenkins and GitHub Actions?**
> | Feature | Jenkins | GitHub Actions |
> |---|---|---|
> | Hosting | Self-hosted | Cloud (GitHub) |
> | Setup | Complex | Simple |
> | Cost | Infrastructure cost | Free tier available |
> | Plugins | 1800+ | 15,000+ Actions marketplace |
> | Best for | Enterprise, complex workflows | GitHub-hosted projects |

---

**Q102. How do you implement zero-downtime deployment with Jenkins?**
> Use **rolling update** or **blue-green** strategy:
> 1. Build and tag new Docker image
> 2. Push to registry
> 3. `kubectl rolling-update` or update Helm chart
> 4. Kubernetes gradually replaces old pods with new ones
> 5. Health checks ensure traffic only goes to healthy pods

---

**Q103. What is the `lock` step in Jenkins?**
```groovy
lock('deploy-staging') {
    // Only one build can hold this lock at a time
    sh './deploy-to-staging.sh'
}
```
> Uses the **Lockable Resources Plugin** to prevent concurrent access to shared resources.

---

**Q104. How do you dynamically generate pipeline stages?**
```groovy
def services = ['auth-service', 'payment-service', 'notification-service']

pipeline {
    agent any
    stages {
        stage('Deploy Services') {
            steps {
                script {
                    def parallelStages = [:]
                    services.each { service ->
                        parallelStages[service] = {
                            sh "kubectl rollout restart deployment/${service}"
                        }
                    }
                    parallel parallelStages
                }
            }
        }
    }
}
```

---

**Q105. What is "Pipeline Durability" in Jenkins?**
> It controls how often Jenkins writes pipeline state to disk:
> - `MAX_SURVIVABILITY` — writes after every step (slow but survives crashes)
> - `PERFORMANCE_OPTIMIZED` — writes less frequently (faster, default)
> - `SURVIVABLE_NONATOMIC` — balance between the two

---

**Q106. How do you implement approval gates for production deployments?**
```groovy
stage('Production Approval') {
    steps {
        script {
            def approver = input(
                message: 'Approve production deployment?',
                submitter: 'lead-dev,devops-team',
                parameters: [string(name: 'REASON', description: 'Approval reason')]
            )
            echo "Approved by ${approver}"
        }
    }
}
```

---

**Q107. How do you handle multiple environments (dev/staging/prod) in Jenkins?**
```groovy
pipeline {
    parameters {
        choice(name: 'DEPLOY_ENV', choices: ['dev', 'staging', 'prod'], description: 'Target environment')
    }
    stages {
        stage('Deploy') {
            steps {
                script {
                    if (params.DEPLOY_ENV == 'prod') {
                        input 'Confirm production deployment?'
                    }
                    sh "ansible-playbook deploy.yml -e env=${params.DEPLOY_ENV}"
                }
            }
        }
    }
}
```

---

**Q108. What monitoring do you set up for Jenkins itself?**
> 1. **Monitoring Plugin** — exposes JVM and Jenkins metrics
> 2. **Prometheus Plugin** — expose metrics for Prometheus scraping
> 3. Alert on: build queue depth, agent offline, failed builds, disk space, memory usage
> 4. Dashboard in Grafana showing build success rate, build duration trends

---

**Q109. How do you handle Jenkins upgrades safely?**
> 1. Test upgrade in a non-production Jenkins instance first
> 2. Take full backup of `$JENKINS_HOME`
> 3. Review release notes and plugin compatibility
> 4. Upgrade plugins before upgrading Jenkins core
> 5. Schedule during low-traffic window
> 6. Have rollback plan (restore from backup)

---

**Q110. Real-world scenario: Design a complete CI/CD pipeline for a Java microservice.**
> ```
> 1. TRIGGER: Push to GitHub → Webhook fires
> 2. CHECKOUT: Pull code from GitHub
> 3. BUILD: mvn clean package
> 4. UNIT TEST: mvn test (JUnit reports published)
> 5. CODE QUALITY: SonarQube scan (Quality Gate check)
> 6. SECURITY SCAN: OWASP dependency check
> 7. DOCKER BUILD: Build image, tag with BUILD_NUMBER
> 8. PUSH IMAGE: Push to ECR/Docker Hub
> 9. DEPLOY DEV: kubectl apply to dev namespace
> 10. INTEGRATION TESTS: Run against dev environment
> 11. [MANUAL GATE]: Approve staging deployment
> 12. DEPLOY STAGING: kubectl apply to staging
> 13. SMOKE TESTS: Quick sanity checks
> 14. [MANUAL GATE]: Approve production
> 15. DEPLOY PROD: Rolling update in production
> 16. HEALTH CHECK: Verify pods are running
> 17. NOTIFY: Slack message with build info
> ```

---

## 📅 Study Schedule

| Day | Topic | File |
|-----|-------|------|
| **Day 1 (Today)** | ✅ Jenkins | `Day1_Jenkins_Interview_QA.md` |
| Day 2 | AWS | `Day2_AWS_Interview_QA.md` |
| Day 3 | Docker | `Day3_Docker_Interview_QA.md` |
| Day 4 | Kubernetes | `Day4_Kubernetes_Interview_QA.md` |
| Day 5 | Terraform | `Day5_Terraform_Interview_QA.md` |
| Day 6 | Linux | `Day6_Linux_Interview_QA.md` |
| Day 7 | Git | `Day7_Git_Interview_QA.md` |
| Day 8 | Ansible | `Day8_Ansible_Interview_QA.md` |
| Day 9 | Monitoring (Prometheus/Grafana) | `Day9_Monitoring_Interview_QA.md` |
| Day 10 | Revision + Scenario Questions | `Day10_Revision_Scenarios.md` |

---

## 💡 Quick Tips for Interview

- **Always give real examples** from your experience: *"In my project, we used Jenkins with..."*
- **Draw diagrams** when explaining architecture (Master-Agent, CI/CD flow)
- **Know the why** — not just what Jenkins does, but WHY you chose it
- **Common mistakes to avoid:** Running builds on master, storing secrets in code, not using pipelines
- **Trending topics:** Jenkins on Kubernetes, GitOps, Jenkins X, migration to GitHub Actions

---

*Good luck with your interview! 🎯 Come back tomorrow for Day 2: AWS*
