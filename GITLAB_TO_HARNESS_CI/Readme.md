# Migration Guide: Gitlab to Harness CI
## Introduction

GitLab CI/CD and Harness CI both allow you to create workflows that automatically build, test, publish, release, and deploy code. 
GitLab CI/CD and Harness CI share some similarities in workflow configuration:

- Workflow configuration files are written in YAML and are stored in the code's repository.
- Workflows include one or more jobs/stages.
- Jobs/Stages include one or more steps or individual commands.

There are a few differences, and this guide will show you the important differences so that you can migrate your workflow to Harness CI.

## Key Differences 

- Gitlab configurations are stored in the ```.gitlab-ci.yml``` file in the root directory of your repository while Harness provides inline pipeline storage 
or importing pipeline YAML (Pipeline-as-Code) from the Git option.

- **Testing and Verification:** GitLab CI comes with a set of test suites that apply to any language. 
Harness CI goes beyond that. Test Intelligence is one of Harness CI's most advanced features which dramatically reduces build times. 
It does so by running incremental builds which allow Harness to run relevant tests and bypass unnecessary tests. 

- **Secrets Management:** GitLab does not offer native secrets management capabilities. They have selected Vault by HashiCorp as their first 
supported secrets management partner which means you must first configure your Vault server. Harness includes a built-in Secrets Management 
feature that enables you to store encrypted secrets, such as access keys, and use them in your Harness applications.

## Comparison

**1. Define a Stage that executes a single build step.**

In GitLab, jobs are a fundamental element in the configuration file. The repositories are automatically fetched in Gitlab CI.

- Gitlab

      job1:
        script: "execute-script-for-job1"
        
- Harness CI

      stages:
        - stage:
          name: build test and run 


**2. Docker image definition.**

Gitlab CI defines images at the job level and supports setting this globally to be used by all jobs that don’t have images defined.

- Gitlab
    
      job1:
        image: ruby:2.6
        
        
You add a Docker image to Harness by connecting to your repo using a [Harness Artifact Server](https://docs.harness.io/article/7dghbx1dbl-configuring-artifact-server), and then by entering its name in a 
Harness Service's Artifact Source Docker Image Name setting.

- Harness CI

       - stage:
        name: stage1 
        identifier: stage1
        type: CI
        spec:
          cloneCodebase: true # Connector clones the repository 
          execution:
            steps:
              - step:
                  type: Run
                  name: step1
                  identifier: step1
                  spec:
                    connectorRef: Dhruba-Connector
                    image: node:17.2.0 #the primary container, where your job's commands are run
                    shell: Bash
                    command: echo "successful" # run the `echo` command
                    

**3. Stages & Steps in Harness CI vs Jobs & Stages in Gitlab.**

Jobs on the same stage run in parallel, and only run after previous stages complete. 
Execution of the next stage is skipped when a job fails by default.

- Gitlab

      stages:
        - build
        - test
        - deploy

             job1:
             stage: build
             script: make build dependencies

             job2:
             stage: build
             script: make build artifacts

             job3:
             stage: test
             script: make a test

             job4:
             stage: deploy
             script: make deploy
             environment: production



Harness CI allows you to create complex pipelines by breaking them down into Stages. 
You can create multiple Stages by combining various CI Steps. Each Stage includes Steps for building, pushing, and testing your code. 
The Codebase configured in the first stage can be imported into subsequent stages. This reduces redundancy and simplifies pipeline creation.

- Harness CI

                      stages:
                        - stage:
                             name: build test and run
                             identifier: build_test_and_run
                             description: ""
                             type: CI
                             spec:
                              cloneCodebase: true
                              execution:
                              steps:
                                     - step:
                                       type: Run
                                       name: install node modules
                                       identifier: install_node_modules
                                       spec:
                                          shell: Sh
                                          command: npm install
                                     - step:
                                       type: Run
                                       name: create the image
                                       identifier: create_image
                                       spec:
                                          shell: Sh
                                          command: |-
                                          touch nodejsdockerfile
                                          cat > nodejsdockerfile <<- EOM
                                          FROM node:14
                                          WORKDIR /nodejshelloworld
                                          COPY package*.json index.js ./
                                          RUN npm install
                                          EXPOSE 8080
                                          CMD ["node", "index.js"]
                                          EOM
                                          cat nodejsdockerfile
                                     - step:
                                       type: BuildAndPushDockerRegistry
                                       name: build and push an image to docker
                                       identifier: build_and_push_an_image_to_docker
                                       spec:
                                          connectorRef: <+input>
                                          repo: <+input>
                                          tags:
                                           - latest
                                          dockerfile: Dockerfile
                                          platform:
                                          os: Linux
                                          arch: Amd64
                                          runtime:
                                          type: Cloud
                                          spec: {}
                                          
                                          

**4. Scheduled run in Gitlab vs Scheduling pipelines using Triggers in Harness CI.**

GitLab CI/CD has an easy-to-use UI to schedule pipelines. Also, rules can be used to determine if jobs should be included or excluded from a scheduled 
pipeline.

- Gitlab

              job1:
                script:
                   - make build
                rules:
                   - if: $CI_PIPELINE_SOURCE == "schedule" && $CI_COMMIT_REF_NAME == "try-schedule-workflow"


- Harness CI


             trigger:
                  name: make build
                  identifier: make_build
                  enabled: true
                  tags: {}
                  orgIdentifier: default
                  projectIdentifier: DhrubaCI
                  pipelineIdentifier: Demo3
                  source:
                  type: Scheduled
                              spec:
                              type: Cron
                              spec:
                              expression: 45 13 13 09 2
                              inputYaml: |
                  pipeline:
                      identifier: Demo3
                      properties:
                           ci:
                                codebase:
                                build:
                                type: branch
                                spec:
                                      branch: main
                                      
                                      
                                      
                                      
**5. Matrix**   

GitLab provides a method to make clones of a job and run them in parallel for faster execution using the ```parallel:``` keyword.

- Gitlab 


            .run-test:
             script: run-test $PLATFORM
             stage: test

                  test-win:
                  extends: .run-test
                  variables:
                      - PLATFORM: windows
                  test-mac:
                  extends: .run-test
                  variables:
                      - PLATFORM: mac
                  test-linux:
                  extends: .run-test
                  variables:
                     - PLATFORM: linux


Matrix is 1 of 2 looping executing strategies provided by Harness. 
Matrix gives the ability to execute the same set of tasks multiple times for a bunch of different configurations. 
This is achieved by mentioning user-defined tags and referencing them in the pipeline using the following expression:

```<+matrix.usertag>```

[Check out the documentation here](https://docs.harness.io/article/eh4azj73m4#matrix)

- Harness CI


                    stages:
                      - stage:
                        name: Stage1
                        identifier: Stage1
                        type: CI
                        spec:
                        cloneCodebase: true
                        execution:
                          steps:
                                - step:
                                  type: Run
                                  name: step1
                                  identifier: step1
                                  spec:
                                      connectorRef: Dhruba-Docker
                                      image: alpine
                                      shell: Bash
                                      command: echo “Testing on  <+matrix.testparam>”
                                  strategy:
                                  matrix:
                                      testparam:
                                              - node
                                              - python
                                              - ubuntu
                                                maxConcurrency: 3
                                                
                                                

**6. Environment Variables**

 In GitLab, CI/CD variables can be defined by going to Settings » CI/CD » Variables or by simply defining them in the ```.gitlab-ci.yml``` file.
 
- Gitlab
 
        test_variable:
        stage: test
           script:
           - echo "$CI_JOB_STAGE"
        variables:
        TEST_VAR: "All jobs can use this variable's value"

        job1:
        variables:
        TEST_VAR_JOB: "Only job1 can use this variable's value"
        script:
           - echo "$TEST_VAR" and "$TEST_VAR_JOB"

  
- Harness CI

              variables:
                  - name: BUILD_PURPOSE
                    type: String
                    description: ""
                    value: RELEASE
                    
                                       
**7. Build and Push to Docker Registry**

- Gitlab

              build image:
                image: docker:20.10.10
              services:
                 - docker:20.10.10-dind
              rules:
                 - if: $CI_PIPELINE_SOURCE == "schedule"        
              script:
                 - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER $CI_REGISTRY --password-stdin
                 - docker build -t $CI_REGISTRY_IMAGE .
                 - docker push $CI_REGISTRY_IMAGE
        
              run tests:
              image: registry.gitlab.com/somegroup/some-image-name
              rules:
                - if: $CI_PIPELINE_SOURCE != "schedule"    
              script:
                - python --version
                - pip --version
                - pytest --version


- Harness CI

             - step:
                  type: BuildAndPushDockerRegistry
                  name: build and push to the docker registry
                  identifier: build_and_push_to_the_docker_registry
                  spec:
                  connectorRef: Dhruba-Docker
                  repo: dhrubajyotichakraborty/sample-app-test
                  tags:
                    - latest
                  dockerfile: Dockerfile
                  
                  
                  
## Triggers

**Harness CI**

Harness supports Webhooks triggers, Artifacts triggers, Manifest triggers, and scheduled triggers.  
The two most commonly used triggers are webhook triggers based on Git events and Scheduled Triggers based on corn expression. 
To know more about creating a trigger visit  [Harness Triggers](https://docs.harness.io/category/oya6qhmmaw-trigger-category)

**Gitlab CI Trigger: Through API**

Webhooks are a convenient way to trigger CI/CD on demand by sending an HTTP post request to specialized URLs. 
This is particularly useful for event-based triggering, in which a webhook can be called whenever a specified event occurs.

## Auto DevOps  and Harness Plugin and Step Template  

**Harness-CI**

Harness has Plugin Step which is Docker Containers to perform a predefined task. Read more about Harness Plugins [here](https://docs.harness.io/article/8r5c3yvb8k-plugin-step-settings-reference) 
Harness also enables you to standardize and create step templates that can be reused across Pipelines and teams that use Harness. 
Read More about Step Template [Here](https://harness.atlassian.net/wiki/spaces/CME/pages/21227110880/Migration+Guide+-+GitLab+to+Harness+CI)

**Gitlab**

[Auto DevOps](https://about.gitlab.com/stages-devops-lifecycle/auto-devops/) is a GitLab-exclusive feature that provides predefined CI/CD configurations that automatically detect, build, test, deploy, and monitor 
your applications. Rather than just accessing a template Auto DevOps is a setting within your GitLab instance that is [enabled by default.](https://harness.atlassian.net/wiki/spaces/CME/pages/21227110880/Migration+Guide+-+GitLab+to+Harness+CI)

**We recommend going through the below list to get you more comfortable before going ahead with complex config migration.** 

- [Caching](https://docs.harness.io/category/01tyeraya4-caching-ci-data)
- [Platform Concepts](https://docs.harness.io/category/sy6sod35zi-platform-concepts) 
- [Platform How To’s](https://docs.harness.io/category/uepsjmurpb-platform-how-tos)
