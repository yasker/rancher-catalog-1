# Rancher Pipeline

Easier to use, Easier to integrate CI/CD with Rancher.

## Dependencies

- jenkins-master: Based on jenkins:2.60.2 wrapped in dind container.
- jenkins-boot: Store the setup information like initial admin user and plugins 
  - Plugins list (see `/pluginManager/installed` after deploy jenkins-master)
  - Default user (User: admin, Password: admin)
- pipeline-server: The main service for connecting rancher and jenkins-master, and maintaining the major logic of CI/CD.

## Concept

Basic Concept of Rancher Pipeline

### Pipeline

Pipeline is a set of configuration to define how a pipeline running. Pipeline must be starting with SCM.
There two Critical concepts here -- `Stage` and `Step`

#### Stage

A `Stage` is a group of `Step` used to organize your pipeline. You can give it a name like `Test`, `Build`, etc. Further more, you can configure `approvers` on a `Stage` so that `Activity` will be pending when running into this stage, after `approver` approve this `Stage`, `Activity` will continue to run. 

>**Note:** you can't configure name or approvers on the first stage, because we design it as an initial step to checkout source code.

#### Step

A `Step` is a minimum execution unit. There are many types of `Step` to provide different CI/CD features. For more details, see [Steps](#step-types)

#### Status
- ##### Active
An active `Pipeline` means the cron task and webhook is enabled.

- ##### Inactive
An inactive `Pipeline` means the cron task and webhook is disabled. you can just run a pipeline manually. 

### Activity

An `Activity` is an execution of `Pipeline`. It contains the result of execution. like Start Time, End Time, Git commit, Logs, etc.

#### Status

- ##### Building
Activity is running.

- ##### Pending
Activity is waitting for approving.

- ##### Denied
Activity has been denied.

- ##### Success
Activity succeed.

- ##### Fail
Activity failed.

## Guide

Deploy `Rancher Pipeline` to your Rancher Environment, and add a pipeline to automatically build, test and deploy your code.

### Deploy

#### Requirements
- cattle environment
- [Library catalog](http://rancher.com/docs/rancher/v1.6/en/catalog/)

Launch App called CICD from [Library catalog](http://rancher.com/docs/rancher/v1.6/en/catalog/).

When CICD is deployed, there will be a tab called Pipeline show in the top menu.

Click tab Pipeline to use our CICD.

### Create a Pipeline

After Navigate to Pipeline tab on top menu, there will be two tabs called `Activities` and `Pipeliens`. Click `Pipelines`. Then Click `Add Pipeline` on the top-right corner of `Pipeline table`. You have to start with the SCM stage in which git repository is required. After configure your git repository, you can start to configure your pipeline using different [Stypes](#step-types).

#### Set approvers

On every stage except the first one, you can authorize some users to approve it. When `Activity` produced by this Pipeline progress to the stage need to be approved, The `Activity` will be pending until someone approve/deny it.

#### Step types

- ##### SCM
SCM(Source Code Management) will only available on the first stage. This stype is used to checkout git repository.

- ##### Task
Task is the most fundamental type, and yet is the most powerful type. you can use any image you want to excute the CI.
>**As a Service:** When you check `As a Service` checkbox, it means this container will keep running until the whole pipeline finish. This is very useful when you want to use a database as a test db. For more details see [Run a mysql server](#run-a-mysql-server)

- ##### Build
Build is used to build a image from dockerfile which come from your source code or the file you upload. you can config it in the UI. Also you can use [Environment Variables](#environment-variables) to tag your image

- ##### Upgrade Service
Upgrade Service is used to upgrade a service in any cattle environment, the default target environment is the environment CICD deployed, you can target other environments by providing `Environment API Key`. You can use [Environment Variables](#environment-variables) to access the image build from `Build` stype.

- ##### Upgrade Stack
Upgrade Stack is used to upgrade a stack in any cattle environment, the default target environment is the environment CICD deployed, you can target other environments by providing `Environment API Key`. You can use [Environment Variables](#environment-variables) to access the image build from `Build` stype. 

- ##### Upgrade Catalog 


#### Set Cron Task

you can set cron task to run pipeline at intervals. Also, you can configure the cron TimeZone to your local timezone or others.
<!-- TODO:  -->

### Run Pipeline

After create a Pipeline, you can run it in the Pipelines table by clicking the action menu, finding the `Run` button. Also, you can run multiple Pipeline in the Activities tab, by clicking `Run pipeline` button, and choosing the pipeline you want to run.

#### Activity Logs

After run a pipeline, there will be an acitivity produced. you can access the logs in the detail page. And, you can access the console log of every step by clicking the step name under the Record.

### Environment Variables

| NAME                           | DESC                |
|--------------------------------|---------------------|
| GIT_PREVIOUS_COMMIT            | previous commit sha |
| GIT_COMMIT                     | commit sha          |
| GIT_PREVIOUS_SUCCESSFUL_COMMIT |                     |
| GIT_BRANCH                     |                     |
| GIT_LOCAL_BRANCH               |                     |
| GIT_URL                        |                     |
| GIT_COMMITTER_NAME             |                     |
| GIT_AUTHOR_NAME                |                     |
| GIT_COMMITTER_EMAIL            |                     |
| BUILD_NUMBER/ID                |                     |
| PIPELINE_NAME                  |                     |
| TRIGGER_TYPE                   |                     |
| NODE_NAME                      |                     |
| ACTIVITY_ID                    |                     |
| ACTIVITY_SEQUENCE              |                     |
| ACTIVITY_NODENAME              | activity            |

### Backup/Restore

The CICD data are stored in two separate place, the whole pipeline and basic activity status informations are stored in the `generic object` of rancher/server's database, the detailed console log of activity is stored in jenkins-master. So, if you want to backup the whole app data, you have to back up these two data.

#### Backup/Restore Jenkins data

If you want to backup the Jenkins data, just copy the Volume of jenkins-boot into your backup directory. When you want restore these data, just use your backup directory as Volume of CICD.

## Progressive Use Case

### Github webhook

### Run script with task

### Run a mysql server

### Build/Push Image

### Set Approvers

### Upgrade Service

### Upgrade Stack

### Upgrade Catalog