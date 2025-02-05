# Jenkins GitHub Integration Guide

## Table of Contents

- 1. [Benefits of CI/CD (Continuous Integration & Continuous Deployment)](#benefits-of-cicd-continuous-delivery--continuous-deployment)
  - 1.1 [Continuous Integration (CI)](#continuous-integration-ci)
  -  1.2 [Continuous Deployment (CD)](#continuous-development-cd)
   [Setting Up Jenkins](#setting-up-jenkins)
2. [Creating a Freestyle Project](#creating-a-freestyle-project)
3. [Running and Building Jobs](#running-and-building-jobs)
4. [Creating a Multi-Stage Pipeline](#creating-a-multi-stage-pipeline)

---

## Benefits of CI/CD (Continuous Delivery & Continuous Deployment)

- ✅ Less Work, More Automation
CI/CD reduces manual effort by automating builds, tests, and deployments—giving developers and IT teams more time to focus on innovation.

- ⚡ Faster Delivery, More Business Value
Ship new features, bug fixes, and improvements at the click of a button. Users get the latest code faster, maximising business impact.

- 🔒 Reduced Risk, More Stability
Frequent small changes minimise the risk of breaking the codebase. Automated build, test, and deployment processes ensure consistency and reduce human error.

CI/CD isn't just a workflow... it's a smarter way to build, test, and deploy! 🚀

### Continuous Integration (CI)

**Branching Strategy: Feature Branch → Dev Branch → Main Branch**

- Developers push changes to a feature branch, triggering an automated build and test process via CI.
- If the CI pipeline succeeds, the changes are merged into the dev or main branch.
- CI is triggered frequently on feature branch pushes to ensure code integrity.
A-  webhook (e.g., to Jenkins) notifies the CI server of code changes, initiating the necessary CI jobs.
- In our case, we are not building a containerised application or a custom Azure image—we are simply testing new application code. If it passes, it is merged into the main branch.

### Continuous development (CD)

- Once CI completes successfully, CD automatically deploys the tested code to the production environment.
- In our case, deployment is to a VM, though in many setups, the CI output is a Docker container, deployed to a Kubernetes cluster with multiple worker nodes.
- When multiple developers work on their own feature branches, it's crucial to stay synced with the latest version of the codebase.
- After testing, the verified code is deployed to production, ensuring only stable updates reach end users.
- Once in production, new features and updates are immediately available for users.
---

## Setting Up Jenkins

1. Start Jenkins and navigate to the web interface (`http://52.31.15.176:8080/` by default).
2. Unlock Jenkins by using your password..
3. Install the recommended plugins.
4. Create an admin user and complete the setup.

## Creating a Freestyle Project

1. Go to the **Jenkins Dashboard**.
2. Click **New Item**.
3. Enter a project name and select **Freestyle project**.
4. Click **OK** to create the project.
5. In the configuration page, scroll down to **Build Steps**.
6. Click **Add build step** > **Execute shell**.
7. Enter the necessary shell commands for your build process.
8. Click **Save**.

---

## Running and Building Jobs

1. On the **Dashboard** Open **New item** then select **Freestyle Project**.
2. In the **Build steps** section select **command > execute shell** and enter your code.
3. Monitor the progress in the **Build History** panel.
4. Click on a build number to view detailed logs.
5. Turn your project into a muli-stage build but selecting ***Post Build Actions > Build Other Project**
```
Console outcome should say started from...
```
# ENTER-SCREENSHOT-HERE
---

## Creating a Multi-Stage Pipeline

1. Go to the **Jenkins Dashboard**.
2. Click **New Item**.
3. Enter a project name and select **Pipeline**.
4. Click **OK** to create the pipeline.
5. Scroll down to the **Pipeline** section.
6. Choose **Pipeline script** and enter your multi-stage pipeline configuration (example below would output the time):
```
time
```

