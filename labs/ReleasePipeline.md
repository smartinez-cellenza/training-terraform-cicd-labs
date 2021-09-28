# Create a release pipeline

## Lab overview

In this lab, you will learn how to use release pipeline.

## Objectives

After you complete this lab, you will be able to:

-   Create a release pipeline using yaml
-   Deploy Terraform template using CI/CD

## Instructions

### Before you start

- Check your access to the Azure Subscription and Resource Group provided for this training.
- Check your access to the Azure DevOps Organization and project provided for this training.
- Project has branch configuration according to the lab Manage Terraform In Azure Repo Git

### Exercise 1: Create Release

In the Azure portal, select your project

Select your project

In the Pipelines blade, select Release

Click on **New Pipeline**

Select **Start with an empty job**

In the stage window, rename the stage **dev** and close the window

Click on **Add an artifact

- Source Type : **Azure Repository**
- Project : **your project**
- Default branch : **dev**

![repo_git](../assets/release_artifact.PNG)

In the **dev** change the agent specification to use ubuntu-20.04

![repo_git](../assets/stage_agent.PNG)

In the variable tab, create variables

![repo_git](../assets/release_variable.PNG)

- ARM_SUBSCRIPTION_ID : Id of the provided subscription
- ARM_ACCESS_KEY : An access key of the backend Storage Account
- TF_VAR_admin_account_password : Admin account password of the DB

Add a new Azure CLI task

![repo_git](../assets/stage_job.PNG)

Configure this task



Select **Approvals**

In the Approver list, add your user

> All Deployment job regarding this environment should now be validated


### Exercice 2: Add this pipeline to the policies on main branch

Go to the project settings -> Repositories

Select the terraform-sample project

Select the policies blade

In the Branch Policies, select the main branch

Add a new Build validation

![repo_git](../assets/build_branch_policy.PNG)

Leave the default options

### Exercice 2: Create a pull request from dev into main

In the Azure Repo blade, select the **terraform-sample** repo

In the repository blade sub-menu, select Pull Requests

Create a new Pull request from **dev** into **main**

> Notice the build validation is triggered