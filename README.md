# Azure Pipelines configuration [![Build Status](https://dev.azure.com/x42martinez/demo-pipelines/_apis/build/status/txabman42.azure-pipelines-cicd?repoName=txabman42%2Fazure-pipelines-cicd&branchName=master)](https://dev.azure.com/x42martinez/demo-pipelines/_build/latest?definitionId=2&repoName=txabman42%2Fazure-pipelines-cicd&branchName=master)

This repository stores the configuration of the Azure pipelines used in the azure-pipelines-cicd repository. It is a demo repository in order to integrate a complete flow of Continuous Integration (CI) using Azure DevOps Pipelines.

An attempt has been made to follow [Code based configuration](https://rollout.io/blog/configuration-as-code-everything-need-know/), which gives the possibility of having advantage of version control and simplifies the configuration data.

## Flow description
There is only a single pipeline that will contain all the necessary stages. This pipeline will be made up of the following stages:
- stage-maven-build
- stage-maven-publish
- stage-maven-github-release

### Parameters
The parameters of the pipeline are as follows:
- poolName
- poolDemands
- mavenSettingsFile
- artifactServiceName
- githubServiceName

An *Agent Pool* has been created for this specific project. In this way, we can use our virtual machines to execute the pipeline processes.

![Agent pool](https://i.ibb.co/D5GPCDZ/Screenshot-2020-07-20-at-21-32-09.png)

A virtual machine on Google Cloud Platform (GCP) has been used as the primary agent to run all flows. This helps to undock us from Azure and will leave the environment in which to execute these tasks to our choice. In addition, it gives us full control over the machine, giving the ability to install all the necessary tools without any problem.

![Self-hosted agent](https://i.ibb.co/x2dWYVT/Screenshot-2020-07-21-at-08-00-20.png)

### Stages
There are three main stages:
- **stage-maven-build**: It is responsible for compiling and executing all maven tests.
- **stage-maven-publish**: Publish the generated maven artifact to an AWS S3. In this way, we will always have all the artifacts, both snapshot and release, stored in a private storage.
- **stage-maven-github-release**: Create a re-release on github based on the tag of the last commit.

### Workflow
The compilation and execution of tests will always be carried out on all pipelines. However, artifacts will only be published when there have been changes in one of the following branches:
- release
- master

![Pipeline executions](https://i.ibb.co/ZMNW2xh/Screenshot-2020-07-21-at-08-07-44.png)

Finally, the release will only be created when changes are made to the master branch and the last commit contains a tag.

All this will generate three different cases:

- **Pull requests (PR)**: Only the stage * stage-maven-build * will be executed. By preventing it from merging until the pipeline is executed properly, we prevent changes to develop or master from merging.

![PR pipeline](https://i.ibb.co/PDZ7T4n/Screenshot-2020-07-21-at-08-08-09.png)

- **Merge into develop**: Once the PR to develop is accepted, the pipeline will be launched in this branch where the maven-build phase will be carried out and the artifact will be published to the corresponding repository.

![Develop pipeline](https://i.ibb.co/9vdgN4b/Screenshot-2020-07-21-at-08-08-26.png)

- **Merge into master**: In the latter case, in addition to everything done by merging develop, a release will be created on GitHub with the latest version.

![Master pipeline](https://i.ibb.co/7zfSpNy/Screenshot-2020-07-21-at-08-08-43.png)

![GitHub Release](https://i.ibb.co/y4hTnsx/Screenshot-2020-07-21-at-08-22-43.png)
