# Create a build pipeline

## Lab overview

In this lab, you will learn how to use Build pipeline.

## Objectives

After you complete this lab, you will be able to:

-   Create a build pipeline using yaml
-   Trigger this pipeline when a Pull Request is created and set it mandatory

## Instructions

### Before you start

- Check your access to the Azure Subscription and Resource Group provided for this training.
- Check your access to the Azure DevOps Organization and project provided for this training.
- Project has branch configuration according to the lab Manage Terraform In Azure Repo Git

### Exercise 1: Create pipeline

Select your *terraform-sample* in Azure DevOps portal

Select the *dev* branch

Create a new folder

![repo_git](../assets/build_create_folder.PNG)

- New folder name : pipelines
- New file name : PR.yml

Copy the following code in the editor

```yaml
# Variable Group 'terraform-DEV' was defined in the Variables tab
trigger: none

jobs:
- job: Linter
  displayName: Linter
  pool:
    vmImage: ubuntu-20.04
  steps:
  - checkout: self
  - task: PowerShell@2
    inputs:
      targetType: 'inline'
      script: |
        cd ./src/terraform
        terraform fmt -recursive -check -diff
```

Commit this file

> Notice the different sections in this yaml file

Go to the Pipelines blade in Azure DevOps and create a new pipeline

![repo_git](../assets/build_new_pipeline.PNG)

In the Where is your source code step, select **Azure Repo Git**

In the Select a repository step, select **terraform-sample**

In the Configure your pipeline step, select **Existing Azure Pipelines YAML file**

In the select an exising yaml file
- select the **dev** branch
- Fill the path : **/pipelines/PR.yml**

Click on Run to execute the pipeline

> Check the pipeline execution

> This pipeline use terraform fmt to lint terraform source code. If you have errors, fix them and run the pipeline again

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