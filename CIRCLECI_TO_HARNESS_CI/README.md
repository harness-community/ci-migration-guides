## CircleCI To Harness Migration

## Key differences

CircleCi provides a Declarative YAML option for pipeline configuration. The Harness has a pipeline studio that provides visual and YAML editors to configure pipelines.

CircleCi configurations are stored in the path `.circleci/config.yml` at the root of your source code repository on the counter. Harness provides inline pipeline storage or importing [Pipeline YAML (Pipeline-as-Code)](https://docs.harness.io/article/q1nnyk7h4v-import-a-pipeline) from the Git option.

## Comparison



<table>
<tr>
<td>  </td> <td> CircleCI </td> <td> Harness CI </td>
</tr>
<tr>
<td>

**Define a Stage that executes a single build step.** 

</td>
<td>

```yaml
jobs:
  job1:
    steps:
      - checkout
      - run: "execute-script-for-job1"
```

</td> 
<td>
    
```yaml
stages:
   - stage:
       name: build test and run
```
</td>
</tr>

<tr>
<td>

**Specify a docker image to use for a job.** 

</td>
<td>

```yaml
jobs:
  hello-job:
    steps:
      - checkout # check out the code in the project directory
      - run: echo "hello CI" # run the `echo` command
    docker:
      - image: cimg/node:17.2.0 # the primary container, where your job's commands are run
        auth:
          username: mydockerhub-user
          password: $DOCKERHUB_PASSWORD 
```

</td> 
<td>
    
```yaml
- stage:
        name: hello-job
        identifier: hellojob
        type: CI
        spec:
          cloneCodebase: true # Connector clones the repositery 
          execution:
            steps:
              - step:
                  type: Run
                  name: hello
                  identifier: hello
                  spec:
                    connectorRef: dockerconnectoref
                    image: node:17.2.0 #the primary container, where your job's commands are run
                    shell: Bash
                    command: echo "hello CI" # run the `echo` command
```
</td>
</tr>
<tr>
<td>


**Define a multi-stage build pipeline**. Stage1 and Stage2 runs concurrently. Once they are done, stage3 gets executed. Once stage3 is done, stage4 runs.<br><br>* Circle ci uses workflows to execute jobs in parallel, sequential, or mixed fashion.<br>    <br><br>* In Harness CI Stages are executed in Order of occurrence in the YAML config. Stages defined under the “- parallel” tag execute in a parallel fashion.

</td>
<td>

```yaml
jobs:
  job1:
    docker:
      - image: cimg/node:17.2.0 
        auth:
          username: mydockerhub-user
          password: $DOCKERHUB_PASSWORD  
    steps:
      - checkout 
      - run: echo "job1"
job2:
    docker:
      - image: cimg/node:17.2.0 
        auth:
          username: mydockerhub-user
          password: $DOCKERHUB_PASSWORD  
    steps:
      - checkout 
      - run: echo "job2"
job3:
    docker:
      - image: cimg/node:17.2.0 
        auth:
          username: mydockerhub-user
          password: $DOCKERHUB_PASSWORD  
    steps:
      - checkout 
      - run: echo "job3"
job4:
    docker:
      - image: cimg/node:17.2.0 
        auth:
          username: mydockerhub-user
          password: $DOCKERHUB_PASSWORD  
    steps:
      - checkout 
      - run: echo "job4"      

workflows:
  version: 2
  jobs:
    - job1
    - job2
    - job3:
        requires:
          - job1
          - job2
    - job4:
        requires:
          - job3
```

</td> 
<td>
    
```yaml
  stages:
    - parallel:
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
                        connectorRef: ronakpatildocker
                        image: node:13.0.0
                        shell: Bash
                        command: echo "Downlaod file in parallel with stage 2 "
        - stage:
            name: Stage2
            identifier: stage2
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
                        connectorRef: ronakpatildocker
                        image: node:13.0.0
                        shell: Bash
                        command: echo "step1"
    - stage:
        name: Stage3
        identifier: Stage3
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
                    connectorRef: ronakpatildocker
                    image: node:13.0.0
                    shell: Bash
                    command: echo "step 1 in stage3 . stage 3 requires stage 1 and 2 "
    - stage:
        name: Stage4
        identifier: Stage4
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
                    connectorRef: ronakpatildocker
                    image: node:13.0.0
                    shell: Bash
                    command: echo "step 1 in stage4 . stage 4 requires stage 3"

```
</td>
</tr>

<tr>
<td>

**Environment variables**<br><br>* [Project-level Environment variables](https://circleci.com/docs/set-environment-variable/#set-an-environment-variable-in-a-project) in circle ci are set using the web app and then referenced in the pipeline. To use Environment variables across multiple projects CircleCI uses Context.<br>    <br><br>* Harness allows users to add Variables on Project, Organization, and Account levels and those can be referenced using the following expression: `<+variable.[scope].[variable_id]>`<br>    <br>    * Account-level reference: `<+variable.account.[var Id]>`<br>        <br>    * Org-level reference: `<+variable.org.[var Id]>`<br>        <br>    * Project-level reference: `<+variable.[var Id]>`<br>        <br><br>Read more about Account, project, and Org level Variables [Here.](https://docs.harness.io/article/f3450ye0ul-add-a-variable)

</td>
<td>

```yaml
jobs:
  job1: 
    steps:
      - run: echo $MY_ENV_VAR
```

</td> 
<td>
    
```yaml
- stage:
        name: Stagename
        identifier: Stagename
        spec:
          execution:
            steps:
              - step:
                  type: Run
                  name: step1
                  identifier: step1
                  spec:
                    command: echo "project var: " <+variable.proj_var>
```
</td>
</tr>

<tr>
<td>

**Matrix**<br><br>Matrix jobs in CircleCI are achieved using parameters and then referencing them in the pipeline using the following expression<br><br>`<< parameters.param >>`<br><br>Matrix is 1 of 2 looping executing strategies provided by Harness. Matrix gives the ability to execute the same set of tasks multiple times for a bunch of different configurations. This is achieved by mentioning user-defined tags and referencing them in the pipeline using the following expression:<br><br>`<+matrix.usertag>`<br><br>To know more about Matrix click [Here](https://docs.harness.io/article/eh4azj73m4#matrix)


</td>
<td>

```yaml
jobs:
  job1:
    docker:
      - image: cimg/node:17.2.0 
        auth:
          username: mydockerhub-user
          password: $DOCKERHUB_PASSWORD  
    steps:
      - checkout 
      - run: << parameters.os >>
      
workflows:
  all-tests:
    jobs:
      - job1:
          matrix:
            parameters:
              os: [node,ubuntu ,python]
```

</td> 
<td>
    
```yaml
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
                    connectorRef: dockerconectorref
                    image: dockerimage:tag
                    shell: Bash
                    command: echo “Testing on  <+matrix.testparam>”
        strategy:
          matrix:
            testparam:
              - node
              - python
              - ubuntu
          maxConcurrency: 3
```
</td>
</tr>

  
    
    
</table>

*   Triggers
    *   Harness CI
        *   Harness supports Webhooks triggers, Artifacts triggers, Manifest triggers, and scheduled triggers. The two most commonly used triggers are webhook triggers based on Git events and Scheduled Triggers based on corn expression. To know more about creating a trigger visit [Harness Triggers](https://docs.harness.io/category/oya6qhmmaw-trigger-category)
    *   CircleCi Orbs
        *   CircleCi supports triggering a pipeline on push to the code repository and Scheduled Triggers. To find out more about CircleCI Triggers visit [Circle CI Triggers](https://circleci.com/docs/triggers-overview/)
*   CircleCi Orbs and Harness Plugin and Step Template
    *   Harness-CI
        *   Harness has Plugin Step which is _Docker Containers_ to perform a predefined task. Read more about Harness Plugins [here](https://docs.harness.io/article/8r5c3yvb8k-plugin-step-settings-reference)
        *   Harness also enables you to standardize and create step templates that can be reused across pipelines and teams that use Harness. Read More about Step Template [Here](https://docs.harness.io/article/99y1227h13-run-step-template-quickstart).
    *   CircleCI
        *   CircleCI orbs are reusable shareable configuration packages that are a combination of jobs, commands, and executors. Read more about orbs [Here](https://circleci.com/docs/orb-concepts/)
*   We recommend going through the following list to make you more comfortable before going ahead with complex config migration.
    *   [Caching](https://docs.harness.io/category/01tyeraya4-caching-ci-data)
    *   [Platform Concepts](https://docs.harness.io/category/sy6sod35zi-platform-concepts)
    *   [Platform How To’s](https://docs.harness.io/category/uepsjmurpb-platform-how-tos)