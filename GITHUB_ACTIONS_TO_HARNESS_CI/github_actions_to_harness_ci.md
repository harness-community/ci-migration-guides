# Migration Guide - Github Actions to Harness-CI

## Introduction
Harness CI and GitHub Actions both allow you to create workflows that automatically build, test, publish, release, and deploy code. Harness CI and GitHub Actions share some similarities in workflow configuration:

*   Workflow configuration files are written in YAML and stored in the repository.
*   Workflows include one or more stages/jobs.
*   Stages include one or more steps or individual commands.
*   Steps or tasks can be reused and shared with the community.

## Key Differences 
* With Harness, there’s no scripting needed and configurations are passed to pipelines securely and in a pragmatic way while Github Actions has third-party actions that you can use as semi-plug-and-play functionality.
* HashiCorp created an action to set up and configure the Terraform CLI in the GitHub Actions workflow. There’s also an action for CloudFormation. Harness provides both infrastructure provisioners with a simpler structure and configuration.
* GitHub Actions does not provide a native Accelerate metrics dashboard whereas Harness has a dashboard specifically for these metrics and allows you to set alerts as needed.
* The YAML file for Github actions is stored in the .github/workflows folder in a repository and for Harness CI it’s stored on the Harness itself and can be created from UI or by importing it from a Git Source.

## Comparison 

### 1. Define a stage that executes a single build step

- Github Actions
 
         jobs:
          build_test_and_run:
            name: build test and run 

- Harness CI 
        
        stages:
        -stage:
           name: build test and run 

### 2. Define a step inside a stage

- Github Actions 

       name: code compilation 
        container:
            image: python:3.10.6-alpine
       run: |
            pyhton -m compileall ./

- Harness CI 
     
         step:
           type: Run
           name: "code compilation "
           identifier: code_compilation
         spec:
            connectorRef: docker_Quickstart
            image: python:3.10.6-alpine
            shell: Sh
            command: python -m compileall ./

### 3. Login into Docker registry 

- Github Actions
       
          - name: login to dockerhub
            uses: docker/login-action@v2
            with: 
               username: {{ secrets.DOCKERHUB_USERNAME }}
               password: {{ secrets.DOCKERHUB_TOKEN }}
    
Here:

1. {{ secrets.DOCKERHUB_USERNAME }} : Docker hub username
2. {{ secrets.DOCKERHUB_TOKEN }}: Docker PAT(Personal Access Token)

- Harness CI 

  In Harness CI we have connectors for login into Docker registry , to know more about connectors visit Connecting to Docker Registry.

  A Connector in Harness is a configurable object that connects to an external resource automatically.

  We reference a Connector in your Pipeline by using its Id in connectorRef.

              step:
                type: Run
                name: "code compilation "
                identifier: code_compilation
                spec:
                   connectorRef: docker_Quickstart    

### 4. Defining an environment varibale 

- Github Actions 

          env:
             BUILD: RELEASE

- Harness CI 

          variables:
           - name: BUILD_PURPOSE
             type: String
             description: ""
             value: RELEASE

### 5. Build and Push to Docker Registry 

- Github Actions

          name: build and push docker image
          uses: docker/build-push-action@v3
          with:
            context:
              file: ./pythondockerfile
              push: true
              tags: user/pythonsample:latest

- Harness CI 

  To know more about Buind and Push to Docker Registry visit **[Build and Push to Docker registry](https://docs.harness.io/article/q6fr5bj63w-build-and-push-to-docker-hub-step-settings)**

           - step:
                type: BuildAndPushDockerRegistry
                name: build and push to the docker registry
                identifier: build_and_push_to_the_docker_registry
                spec:
                  connectorRef: docker_Quickstart
                  repo: krishi0408/pythonsample
                   tags:
                      - latest
                   dockerfile: pythondockerfile

### 6. Checkout Code

- Github Actions 

  In Github Actions we use, actions/checkout@v2 , which is the action that checked out your repository to the computer that runs the action.

           steps:
            - name: Checkout code
              uses: actions/checkout@v2

- Harness CI 

  In Harness CI, we have to create GitHub connector as part of the first step which is basically a configurable objet that connects to an external source automatically. 

   Harness Code Repository Connectors connect your Harness account with your Git platform.

   To know about creating a git connector visit **[GitHub Connector](https://docs.harness.io/article/jd77qvieuw-add-a-git-hub-connector)**.

   ![Clone Codebase](./1.png)

   When enabled in pipeline you can see in the yaml:
          
            stage:
             name: build test and run
             identifier: build_test_and_run
             type: CI
             spec:
               cloneCodebase: true

### 7. Trigger a Workflow 

- Github Actions

   To know about triggering a workflow in GitHub actions visit **[Triggering a workflow in GitHub Actions](https://docs.github.com/en/actions/using-workflows/triggering-a-workflow)** and **[Triggering a workflow event](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)**.

           on:
            pull_request:
            branches:
              - main

- Harness CI 
    

  You can trigger Pipelines in response to Git events that match specific payload conditions you set up in the Harness Trigger.

  For example, when a pull request or push event occurs on a Git repo and your Trigger settings match the payload conditions, a CI Pipeline can execute.

  In Harness , you trigger a workflow using trigger option in the pipeline studio.

  To know more about creating a trigger visit **[Trigger Pipeline using Git event Payload Conditions](https://docs.harness.io/article/10y3mvkdvk-trigger-pipelines-using-custom-payload-conditions)**


### 8. Matrix 
 
- Github Actions 

   A matrix strategy lets you use variables in a single job definition to automatically create multiple job runs that are based on the combinations of the variables. For example, you can use a matrix strategy to test your code in multiple versions of a language or on multiple operating systems.

      jobs:
       example_matrix:
        strategy:
         matrix:
           pyhton: [ 3.10.6-alpine,3.10.4-alpine]
    

To know more about matrix in Github Actions visit **[Matrix in Github Actions](https://docs.github.com/en/actions/using-jobs/using-a-matrix-for-your-jobs)**.

- Harness CI 


       
       strategy:
          repeat:
            items:
              - 3.10.6-alpine
              - 3.10.4-alpine
          maxConcurrency: 2
   
To know more about matrix in Harness CI visit **[Lopping strtategies Overview](https://docs.harness.io/article/eh4azj73m4-looping-strategies-matrix-repeat-and-parallelism)**.


## Complete Example

- Github Actions

    Click on [Github Actions Yaml](.github/workflow/main.yaml).

- Harness CI 

    Click on [Harness CI Yaml](.harness/pipeline.yaml). 



  
       



