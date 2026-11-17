# Practical 3: CI/CD Pipeline with Jenkins and Maven

> **Aim:** Build an automated Jenkins declarative CI/CD pipeline that checks out, compiles, tests, packages, and deploys a Maven project whenever source code changes are detected in GitHub.  
> **Target Project Repository:** [lelnotwellyet/java-maven-demo](https://github.com/lelnotwellyet/java-maven-demo)  
> **Jenkins Pipeline Job Name:** `cicd-pipeline`

---

## Table of Contents
1. [Overview & Prerequisites](#overview--prerequisites)
2. [Step 1: Add Jenkinsfile to GitHub Repository](#step-1-add-jenkinsfile-to-github-repository)
3. [Step 2: Create the Pipeline Job (Q3.1)](#step-2-create-the-pipeline-job-q31)
4. [Step 3: Run the Pipeline & Verify Stage View (Q3.3)](#step-3-run-the-pipeline--verify-stage-view-q33)
5. [Step 4: Demonstrate Automated CI Trigger via SCM Polling (Q3.4)](#step-4-demonstrate-automated-ci-trigger-via-scm-polling-q34)
6. [Pipeline Stages Summary](#pipeline-stages-summary)
7. [Troubleshooting & Windows Notes](#troubleshooting--windows-notes)
8. [Screenshots Checklist](#screenshots-checklist)

---

## Overview & Prerequisites

This practical builds directly upon **Practical 2**:
- **GitHub Repository:** `java-maven-demo` containing `pom.xml`, `App.java`, and `AppTest.java`.
- **Global Tool:** `Maven3` configured under **Manage Jenkins > Tools > Maven installations** with automated installer.
- **Java Environment:** Java 21 JDK properly detected by Jenkins.
- **Plugins Required:**
  - `Pipeline` (Workflow plugin)
  - `Pipeline: Stage View` plugin (for visual stage boxes on the job dashboard)
  - `Git` plugin

---

## Step 1: Add Jenkinsfile to GitHub Repository

1. Open your GitHub repository [lelnotwellyet/java-maven-demo](https://github.com/lelnotwellyet/java-maven-demo).
2. Click **Add file > Create new file**.
3. Name the file: `Jenkinsfile` *(at the root of the repo, exact casing, no extension)*.
4. Paste the declarative pipeline script:

```groovy
pipeline {
    agent any
    tools {
        maven 'Maven3'
    }
    triggers {
        pollSCM('H/2 * * * *')
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'Code checked out from GitHub'
            }
        }
        stage('Build') {
            steps {
                bat 'mvn clean compile'
            }
        }
        stage('Test') {
            steps {
                bat 'mvn test'
            }
        }
        stage('Package') {
            steps {
                bat 'mvn package -DskipTests'
            }
        }
        stage('Deploy') {
            steps {
                bat 'if not exist C:\\deploy mkdir C:\\deploy'
                bat 'copy /Y target\\*.jar C:\\deploy\\'
                echo 'Deployed jar to C:\\deploy'
            }
        }
    }
}
```

5. Commit the new file directly to `main` with commit message: `Add Jenkinsfile for CI/CD pipeline`.

> **Screenshot 1:** `Jenkinsfile` on GitHub repository root.

---

## Step 2: Create the Pipeline Job (Q3.1)

1. Open Jenkins Dashboard (`http://localhost:8080`).
2. Click **New Item** in the left sidebar.
3. Enter the job name: `cicd-pipeline`.
4. Select **Pipeline** project and click **OK**.
5. In the configuration page, scroll down to the **Pipeline** section:
   - **Definition:** Select `Pipeline script from SCM`.
   - **SCM:** Select `Git`.
   - **Repository URL:** `https://github.com/lelnotwellyet/java-maven-demo.git`
   - **Credentials:** `- none -` *(public repository)*.
   - **Branches to build > Branch Specifier:** `*/main`
   - **Script Path:** `Jenkinsfile`
6. Click **Save**.

> **Screenshot 2:** Pipeline configuration page showing **Pipeline script from SCM**, Repository URL, Branch Specifier `*/main`, and Script Path `Jenkinsfile`.

---

## Step 3: Run the Pipeline & Verify Stage View (Q3.3)

1. On the `cicd-pipeline` job page, click **Build Now** on the left menu.
2. The initial build will checkout the `Jenkinsfile` from GitHub, parse the declarative syntax, download Maven dependencies, and execute each stage.
3. Observe the **Stage View** table. You should see 5 green stage boxes:
   - `Checkout` (Green)
   - `Build` (Green)
   - `Test` (Green)
   - `Package` (Green)
   - `Deploy` (Green)
4. Click on `#1 > Console Output`. Verify that the log reaches:
   ```text
   [Deploy] Deployed jar to C:\deploy
   ...
   Finished: SUCCESS
   ```
5. **Verify Local Deployment Directory:**
   - Open File Explorer or Command Prompt and check `C:\deploy`:
   ```cmd
   dir C:\deploy
   ```
   - The file `java-maven-demo-1.0.jar` should exist in `C:\deploy\`, confirming the artifact deployment succeeded.

> **Screenshot 3:** Stage View displaying all 5 stages in green (`Checkout`, `Build`, `Test`, `Package`, `Deploy`).  
> **Screenshot 4:** Build #1 Console Output ending with `Finished: SUCCESS`.  
> **Screenshot 5:** The `C:\deploy` directory in File Explorer / Command Prompt showing `java-maven-demo-1.0.jar`.

---

## Step 4: Demonstrate Automated CI Trigger via SCM Polling (Q3.4)

> **Important Note:** In a declarative pipeline, the `triggers { pollSCM('H/2 * * * *') }` block registers with Jenkins **after** the first manual build (#1) completes and evaluates the Jenkinsfile. Build #1 must run manually before polling becomes active.

1. Go to your GitHub repository `java-maven-demo`.
2. Navigate to `src/main/java/com/example/App.java` and click the pencil (edit) icon.
3. Update the printed message string, for example:
   ```java
   System.out.println("Hello from Jenkins Maven build! Pipeline auto triggered! 2 + 3 = " + add(2, 3));
   ```
4. Commit the change to the `main` branch with the commit message: `Test automatic CI pipeline trigger`.
5. Return to Jenkins `cicd-pipeline` page and **do not click anything**.
6. Within 2 minutes, Jenkins polls GitHub, detects the new commit hash, and automatically starts **Build #2**.
7. Click on **Build #2**:
   - At the top of the build summary page, verify it explicitly states:  
     `Started by an SCM change`
   - Check the Stage View: All 5 stages execute and turn green.
   - Check `C:\deploy`: The JAR is refreshed with the new build.

> **Screenshot 6:** Build #2 summary page displaying `Started by an SCM change` and the triggered Stage View.

---

## Pipeline Stages Summary

| Stage Name | Command Executed | Purpose / Explanation |
| :--- | :--- | :--- |
| **Checkout** | `checkout scm` | Pulls the latest commit and branch from GitHub into the Jenkins workspace. |
| **Build** | `bat 'mvn clean compile'` | Cleans previous build artifacts and compiles Java source classes in `src/main/java`. |
| **Test** | `bat 'mvn test'` | Runs automated JUnit unit test suites located in `src/test/java`. Build fails if tests fail. |
| **Package** | `bat 'mvn package -DskipTests'` | Bundles compiled classes and resources into `target/java-maven-demo-1.0.jar`. |
| **Deploy** | `bat 'copy /Y target\*.jar C:\deploy\'` | Creates deployment directory `C:\deploy` (if missing) and copies the packaged JAR for distribution. |

---

## Troubleshooting & Windows Notes

- **Windows Batch Shell (`bat` vs `sh`):**
  - Because Jenkins runs on Windows, stages use `bat '...'` commands instead of Linux `sh '...'`.
  - Directory backslashes in Groovy strings must be escaped: `C:\\deploy`.
- **Stage View Missing?**
  - If you only see a plain build history list without visual stage columns, go to **Manage Jenkins > Plugins > Available plugins**, search for **Pipeline: Stage View**, install it, and restart Jenkins (`net stop jenkins && net start jenkins`).
- **Why Poll SCM instead of Webhooks?**
  - Webhooks require GitHub to send an HTTP POST request to your Jenkins server. Because Jenkins is running on a local lab machine (`localhost:8080`), GitHub on the public internet cannot reach `localhost`. SCM Polling (`H/2 * * * *`) reaches out from your PC to GitHub, which works reliably in local network/lab environments.
- **Maven command not recognized in pipeline?**
  - Ensure `tools { maven 'Maven3' }` matches the exact tool name defined under **Manage Jenkins > Tools > Maven installations**.

---

## Screenshots Checklist

- [ ] **1.** `Jenkinsfile` on GitHub repository root
- [ ] **2.** Pipeline Job Configuration (`Pipeline script from SCM`, Git URL, `*/main`, `Jenkinsfile`)
- [ ] **3.** Stage View with all 5 green stages (`Checkout`, `Build`, `Test`, `Package`, `Deploy`)
- [ ] **4.** Build #1 Console Output (`Finished: SUCCESS`)
- [ ] **5.** `C:\deploy` folder showing `java-maven-demo-1.0.jar`
- [ ] **6.** Build #2 showing `Started by an SCM change`
