<img src="https://cdn.prod.website-files.com/677c400686e724409a5a7409/6790ad949cf622dc8dcd9fe4_nextwork-logo-leather.svg" alt="NextWork" width="300" />

# Build a CI/CD Pipeline with AWS

![Build Status](https://img.shields.io/badge/build-passing-brightgreen)
![AWS](https://img.shields.io/badge/AWS-CodePipeline-orange?logo=amazon-aws)
![GitHub Repo](https://img.shields.io/badge/GitHub-Repository-blue?logo=github)
![License](https://img.shields.io/badge/license-MIT-lightgrey)

**Project Link:** [View Project](http://learn.nextwork.org/projects/aws-devops-codepipeline-updated)

**Author:** Bradley Davel  
**Email:** bradley.davel@outlook.com

---

![Image](http://learn.nextwork.org/sparkling_indigo_heroic_bat/uploads/aws-devops-codepipeline-updated_fbdetger)

---

## Introducing Today's Project!

In today's project, I'll be putting together an automated CI/CD pipeline with AWS CodePipeline!

CodePipeline solves these problems by orchestrating your entire workflow, and giving you visibility into the entire process in one place. It can:

✅ Automatically detect code changes in your GitHub repository.
✅ Automatically trigger CodeBuild to build a new project.
✅ Automatically start CodeDeploy with the new build artifacts.
✅ Automatically roll back a change if something fails.

### Key tools and concepts

In this project, I learnt how to integrate GitHub, CodePipeline, CodeBuild, and CodeDeploy into a working CI/CD pipeline. I understood the role of artifacts between stages, the importance of IAM service roles, and how automatic rollback ensures reliability. Most importantly, I saw how a simple code change in GitHub can automatically be tested, packaged, and deployed into production without manual steps.

### Project reflection

This project took me around 25min to complete from start to finish.

⏱️ Time was spent setting up the pipeline stages (Source, Build, Deploy), configuring IAM service roles, and connecting GitHub.

⚙️ Extra time went into preparing the EC2 instance (installing the CodeDeploy agent) and verifying that the deployment worked.

🧪 I also tested the pipeline by pushing a code change and confirming that it automatically flowed through Source → Build → Deploy.

---

## Starting a CI/CD Pipeline

AWS CodePipeline is Amazon’s fully-managed continuous integration and continuous delivery (CI/CD) service.

It lets you automate the steps required to release software—from pulling code out of your GitHub repo, to building/testing it with CodeBuild, to deploying it with CodeDeploy (or other targets like ECS, Lambda, or S3).

CodePipeline offers different execution modes based on how you want multiple pipeline runs to behave when new commits arrive:

QUEUED (default):

-Every change triggers a pipeline run.
-If a run is in progress, new runs are queued and executed in order.
✅ Best when you want to ensure every commit is built and tested, even if it means waiting.

SUPERSEDED:

-If a pipeline is already running and a new commit arrives, the in-progress execution is canceled.
-Only the latest commit is kept.
✅ Best when only the most recent state matters (e.g., deploying to a dev/test environment).

PARALLEL (newer mode):

-Multiple pipeline executions can run at the same time.
✅ Best for independent builds (e.g., microservices or fan-out test jobs).

AWS CodePipeline needs a service role (IAM role) so that it can act on your behalf and access other AWS resources securely. Since CodePipeline itself doesn’t have permissions by default, the service role defines what actions it’s allowed to perform across services that your pipeline touches.

![Image](http://learn.nextwork.org/sparkling_indigo_heroic_bat/uploads/aws-devops-codepipeline-updated_gdnhtm)

---

## CI/CD Stages

The three stages I’ve set up in my CI/CD pipeline are Source, Build, and Deploy.

While setting up each part, I learnt about artifact chaining between stages, the importance of default branches for consistent triggers, how CodeBuild interprets buildspec.yml, and how CodeDeploy ensures safe, automated deployments with rollback protection.

CodePipeline organizes the three stages into a visual flow: Source, Build, and Deploy. In each stage, you can see the status (success, failure, or in-progress), the provider used (GitHub, CodeBuild, CodeDeploy), the commit ID that triggered the run, and the execution time. Each stage also links to detailed logs, making it easy to debug issues or confirm that the correct commit was built and deployed.

![Image](http://learn.nextwork.org/sparkling_indigo_heroic_bat/uploads/aws-devops-codepipeline-updated_fbdetger)

---

## Source Stage

In the Source stage, the default branch tells CodePipeline which branch of your source repository to monitor for changes. This is the branch that will automatically trigger the pipeline when updates (commits/merges) are pushed.

Webhook events let CodePipeline automatically start your pipeline whenever code is pushed to your specified branch in GitHub. This is what makes our pipeline truly "continuous" – it reacts to code changes in real-time!

Webhooks are like digital notifications. When you enable webhook events, CodePipeline sets up a webhook in your GitHub repository. This webhook is configured to listen for specific events, such as code pushes to the master branch.

Whenever you push code to the master branch, GitHub sends a webhook event (a notification) to CodePipeline. CodePipeline then automatically starts a new pipeline execution in response to this event. It's a seamless way to automate your CI/CD process!

![Image](http://learn.nextwork.org/sparkling_indigo_heroic_bat/uploads/aws-devops-codepipeline-updated_sergt)

---

## Build Stage

The Build stage sets up a CodeBuild project to compile and test my application. I configured the Source stage to output an artifact named SourceArtifact. The input artifact for the Build stage is this SourceArtifact, because CodeBuild needs the exact code package produced by the Source stage in order to run builds consistently and reliably.

![Image](http://learn.nextwork.org/sparkling_indigo_heroic_bat/uploads/aws-devops-codepipeline-updated_j1k2l3m4)

---

## Deploy Stage

In the Deploy stage, I configured AWS CodeDeploy as the deployment provider. I connected it to my CodeDeploy application (nextwork-devops-cicd) and deployment group (nextwork-devops-cicd-deployment-group), which targets my EC2 instances. The stage uses the BuildArtifact from the previous Build stage as its input. I also enabled automatic rollback on failure to ensure that if something goes wrong during deployment, my application reverts to the last successful version.

![Image](http://learn.nextwork.org/sparkling_indigo_heroic_bat/uploads/aws-devops-codepipeline-updated_m4n5o6p7)

---

## Success!

Since my CI/CD pipeline gets triggered by every commit to the default branch, I tested my pipeline by making a simple but visible change in the application’s source code.

I added the following line into my HTML file:
<p>If you see this line, that means your latest changes are automatically deployed into production by CodePipeline!</p>

The moment I pushed the code change to my GitHub repository, the Source stage in CodePipeline immediately detected the update and triggered a new pipeline execution.

Source stage: Retrieved the latest commit from GitHub, packaged it into SourceArtifact, and displayed the commit ID (dffa7a90) as confirmation.

Build stage: Picked up the SourceArtifact, ran the CodeBuild project using buildspec.yml, and produced a BuildArtifact with the updated HTML file.

Deploy stage: Used AWS CodeDeploy to push the BuildArtifact to my EC2 instance. Since deployment succeeded, the new line appeared in the application.

The commit message under each stage reflects the exact change that triggered the pipeline, making it easy to trace the deployed version back to the source commit.

Once my pipeline executed successfully, I confirmed the deployment by testing the running application on my EC2 instance.

1. I copied the Public IPv4 DNS of my EC2 instance.
2. Pasted it into a new browser tab and pressed Enter.
3. The application loaded and displayed the updated web page, which now included my new line of code:
<p>If you see this line, that means your latest changes are automatically deployed into production by CodePipeline!</p>

Seeing this new line rendered in the browser proved that my pipeline was working end-to-end:

-The commit was detected by the Source stage.
-It was built into an artifact by the Build stage.
-Finally, the Deploy stage successfully rolled out the update to my EC2 instance.

![Image](http://learn.nextwork.org/sparkling_indigo_heroic_bat/uploads/aws-devops-codepipeline-updated_e1f2g3h4)

---

