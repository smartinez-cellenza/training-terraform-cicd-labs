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
- Project has branch configuration according to the lab Manage Terraform In Azure Repo Git and backend configuration is done to match your Storage Account

### Exercise 1: Create Environments

In this exercice, we will create the 3 environments (in Azure DevOps) to match the 3 configuration we have.

In the Azure DevOps portal, go the pipelines blade, and selected Environments.

Click on Create environment.

![repo_git](../assets/environment_create.PNG)

In the New environment window, set the information on the environment

- **Name**: dev *(keep it lowercase)*
- **Description**: Dev environment

Repeat this step for the **uat** and **prod** environment

### Exercice 2: Set approbation

In this exercice we will configure approbation for environment deployment for **uat** and **prod** environments.

Our workflow will be te deploy to dev without confirmation, and require a manual approvers for the 2 others environment.

In the Azure DevOps portal, go the pipelines blade, and selected Environments.

Select the **uat** environment.

In the options menu, select **Approval and checks**

![repo_git](../assets/environment_approval.PNG)

Select **Aprovals**

![repo_git](../assets/environment_approval_next.PNG)

Add your self in the **Approvers** list and click on **Create**

> Notice the others checks available, based on Branch or Business Hours

Repeat this step for the **prod** environment

### Exercice 3: Create libraries

In this exercice we will create a library for each of the environment.

A library might be used to store variables and secrets for an environment.

> An environment configuration might be done in multiple libraries. All of this libraries should follow the environment segregation principle.

In the pipeline blade, Select Library and add a variable group

![repo_git](../assets/library_create.PNG)

In the *Properties*, set the *Variable group name* to **dev**

in the *Variables*, add 3 items:

- **ARM_ACCESS_KEY**: On of the Storage Account Access key. Since we will use a Service Principal to authenticate, this information is required. You can get one from the Storage Account Access keys blade in the Azure Portal.
- **ARM_SUBSCRIPTION_ID**: The subscription Id where resources must be deployed. Use the training subscription ID. You can get it from the Azure Portal
- **admin_account_password**: The Admin Account password for the database to be created. Must be Azure compliant (if you're not inspired, P@ssword01! is fine)


For **ARM_ACCESS_KEY** and **admin_account_password**, set them as secret

![repo_git](../assets/library_detail.PNG)

Repeat the same operation for **uat** and **prod** environment. **ARM_ACCESS_KEY** and **ARM_SUBSCRIPTION_ID** won't change in our case (we will always target the same subscription)

### Exercice 4: Create Release pipeline for dev environment

In the exercice, we will create the Release pipeline.

It will use
- The artefact produced by the build pipeline
- The library we created
- The environment we created

A Service Connection has been created and shared in your project. It contains the information on the Service Principal.

Select your *terraform-sample* in Azure DevOps portal

Select the *dev* branch

Create a new file

- New file name : release_dev.yml

Copy the following code in the editor

```yaml
trigger: none

pool:
  vmImage: ubuntu-latest

resources:
 pipelines:
   - pipeline: build
     source: build

jobs :
  - deployment: deploy_dev
    displayName: Deploy Dev Environment
    environment: dev
    variables:
    - group: dev
    strategy:
     runOnce:
       deploy:
         steps:
           - task: AzureCLI@2
             env:
               TF_VAR_admin_account_password: $(admin_account_password)
             displayName: Deploy Dev Environment
             inputs:
              azureSubscription: 'Terraform Service Principal'
              scriptType: 'pscore'
              scriptLocation: 'inlineScript'
              addSpnToEnvironment: true
              inlineScript: |
                cd $(PIPELINE.WORKSPACE)/build/terraform/terraform
                $env:ARM_CLIENT_ID=$env:servicePrincipalId
                $env:ARM_CLIENT_SECRET=$env:servicePrincipalKey
                $env:ARM_TENANT_ID=$env:tenantId
                terraform init -backend-config='../configuration/dev/backend.hcl'
                terraform apply -var-file='../configuration/dev/var.tfvars' -input=false -auto-approve
```

> Notice the different sections

> Notice the deployment step

Go to the Pipelines blade in Azure DevOps and create a new pipeline

![repo_git](../assets/build_new_pipeline.PNG)

In the Where is your source code step, select **Azure Repo Git**

In the Select a repository step, select **terraform-sample**

In the Configure your pipeline step, select **Existing Azure Pipelines YAML file**

In the select an exising yaml file
- select the **dev** branch
- Fill the path : **/pipelines/release.yml**

Click on Run to execute the pipeline

In the pipeline blade, ensure the pipeline has run.

> Go to the Azure Portal and ensure resource has been created in your Resource Group

Select the release pipeline in the pipeline blade, and rename it to **release_dev**

![repo_git](../assets/build_rename.PNG)

### Exercice 4: Create Release pipeline for uat environment

In the exercice, we will create the Release pipeline.

It will use
- The artefact produced by the build pipeline
- The library we created
- The environment we created

A Service Connection has been created and shared in your project. It contains the information on the Service Principal.

Select your *terraform-sample* in Azure DevOps portal

Select the *dev* branch

Create a new file

- New file name : release_uat.yml

Copy the following code in the editor

```yaml
trigger: none

pool:
  vmImage: ubuntu-latest

resources:
 pipelines:
   - pipeline: build
     source: build

stages:
- stage: plan_uat
  displayName: Run terraform plan on uat environment
  jobs :
    - job: plan_uat
      displayName: Run terraform plan on uat environment
      variables:
      - group:  uat
      steps:
      - checkout: self
      - download: build
      - task: AzureCLI@2
        env:
          TF_VAR_admin_account_password: $(admin_account_password)
        displayName: Run terraform plan on uat environment
        inputs:
          azureSubscription: 'Terraform Service Principal'
          scriptType: 'pscore'
          scriptLocation: 'inlineScript'
          addSpnToEnvironment: true
          inlineScript: |
            cd $(PIPELINE.WORKSPACE)/build/terraform/terraform
            $env:ARM_CLIENT_ID=$env:servicePrincipalId
            $env:ARM_CLIENT_SECRET=$env:servicePrincipalKey
            $env:ARM_TENANT_ID=$env:tenantId
            terraform init -backend-config='../configuration/uat/backend.hcl'
            terraform plan -var-file='../configuration/uat/var.tfvars' -input=false

- stage: apply_uat
  displayName: Run terraform plan on uat environment
  dependsOn: plan_uat
  jobs :
    - deployment: deploy_uat
      displayName: Deploy Uat Environment      
      environment: uat
      variables:
      - group: uat
      strategy:
        runOnce:
          deploy:
            steps:
              - task: AzureCLI@2
                env:
                  TF_VAR_admin_account_password: $(admin_account_password)
                displayName: Deploy Uat Environment
                inputs:
                  azureSubscription: 'Terraform Service Principal'
                  scriptType: 'pscore'
                  scriptLocation: 'inlineScript'
                  addSpnToEnvironment: true
                  inlineScript: |
                    cd $(PIPELINE.WORKSPACE)/build/terraform/terraform
                    $env:ARM_CLIENT_ID=$env:servicePrincipalId
                    $env:ARM_CLIENT_SECRET=$env:servicePrincipalKey
                    $env:ARM_TENANT_ID=$env:tenantId
                    terraform init -backend-config='../configuration/uat/backend.hcl'
                    terraform apply -var-file='../configuration/uat/var.tfvars' -input=false -auto-approve

```

> Notice the different sections

Go to the Pipelines blade in Azure DevOps and create a new pipeline

![repo_git](../assets/build_new_pipeline.PNG)

In the Where is your source code step, select **Azure Repo Git**

In the Select a repository step, select **terraform-sample**

In the Configure your pipeline step, select **Existing Azure Pipelines YAML file**

In the select an exising yaml file
- select the **dev** branch
- Fill the path : **/pipelines/release_uat.yml**

Click on Run to execute the pipeline

> Notice the approbation required

In the pipeline blade, ensure the pipeline has run.

> Go to the Azure Portal and ensure resource has been created in your Resource Group

Select the release pipeline in the pipeline blade, and rename it to **release_uat**

![repo_git](../assets/build_rename.PNG)
