# Extend, Build & Deploy Kabanero Pipelines
Developers that use [Kabanero](https://kabanero.io/) pipelines often times have to extend these pipelines to do certain tasks that do not come
in the out-of-the-box Kabanero pipelines. These tasks may include code coverage, or use third party applications like
[Pact Broker](https://docs.pact.io/pact_broker), [Sonarqube](https://www.sonarqube.org/) or [Artifactory](https://jfrog.com/artifactory/)
to full-fill software requirements. Currently, there are not many methods to manage and version control your Kabanero 
pipelines, and the goal of this repository is to help you get going.

You will learn how to package, host your pipelines in different environments such as Git or Artifactory and use
these pipelines to automate the process of updating the [Kabanero custom resource](https://kabanero.io/docs/ref/general/configuration/kabanero-cr-config.html) 
to a respective host where your Kabanero pipelines exist.

## Table of Contents
  * [Extend, Build & Deploy Kabanero Pipelines](#extend-build--deploy-kabanero-pipelines)
  * [Overview](#overview)
  * [Pre-requisites](#pre-requisites)
  * [How to use artifactory-package-release-update pipeline](#how-to-use-artifactory-package-release-update-pipeline)
  * [How to use clone-storefront-ms-push-repos-to-org pipeline](#how-to-use-clone-storefront-ms-push-repos-to-org)
  * [How to use git-package-release-update pipeline](#how-to-use-git-package-release-update-pipeline)
  * [Create tekton webhook](#create-a-tekton-webhook)
  
# Overview

This repository includes 2 [directories](/pipelines), `experimental`(pipelines that are not production-ready and are considered, proof of concept), and `stable`(pipelines that are production ready). It also includes the following pipelines:

| Stable  | Description  |
|---|---|
| artifactory-package-release-update  |  Compress custom pipelines, upload compressed pipelines to artifactory, and updates the Kabanero Custom Resource |
| clone-storefront-ms-push-repos-to-org | Given a github org name, clone storefront microservices and deploy them to the github org   |
| git-package-release-update | Compress custom pipelines, create a github release, upload compressed pipelines to the release, and update the Kabanero Custom Resource |
| storefront-springboot |  This pipeline contains healthcheck, sonar-scan, pact-broker tasks for spring boot applications |

| Incubator  | Description  |
|---|---|
| cloud-foundry  |  Deploys an application to cloud foundry given a specific namespace using CloudFoundry CLI  |
| deploy-app-ibm-cloud  |  Deploys an application to cloud foundry  given a specific namespace using the IBM CLI |

 
   
# Pre-requisites
* Install the following CLI's on your laptop/workstation:

    + [`docker cli`](https://docs.docker.com/docker-for-mac/install/)
    + [`git cli`](https://git-scm.com/downloads)
    + [`oc cli`](https://docs.openshift.com/container-platform/4.3/welcome/index.html)
    + [`Openshift 4.3.5 with CloudPak for Apps`](https://www.ibm.com/cloud/cloud-pak-for-applications)
    + [`tekton cli`](https://github.com/tektoncd/cli)
    + [`Kabanero 0.6.1`](https://github.com/kabanero-io)
    

# How to use artifactory-package-release-update pipeline
### Pre-reqs
- Fork the [devops-pipelines](https://github.com/ibm-garage-ref-storefront/devops-pipelines) repository
- Deploy [Artifactory](https://github.com/ibm-cloud-architecture/gse-devops/tree/master/cloudpak-for-integration-tekton-pipelines#artifactory) on your Openshift cluster

- Generate an [API Key](https://www.jfrog.com/confluence/display/JFROG/User+Profile). 
- Update Artifactory config map
[artifactory-config.yaml](./pipelines/stable/artifactory-package-release-update/configmaps/artifactory-config.yaml) and update the `artifactory_key`. Once done, run the following
commands:

    ```bash
    oc project kabanero
    cd ./configmaps
    oc apply -f artifactory-config.yaml
    ```
### Steps
1) Go to the [pipelines](./pipelines) directory make any modifications you want to do to any of the pipelines, or include your own.

2) Create your pipeline by running the following command:
    ``` bash
    cd pipelines/experimental
    oc apply --recursive --filename pipelines/experminetal/artifactory-package-release-update/
    ```

3) Go to the dashboard and verify that the `artifactory-package-release-update-pl` has been added to the Tekton dashboard

4) Go to section [Create tekton webhook](#create-a-tekton-webhook) to create your web hook.

5) Go to your forked repository and make a change, and your Tekton dashboard should create a 
new pipeline run as shown below:
![](gifs/artifactory-package-release-update-pl-rn.gif)
Where the `git-source` is defined as the pipeline resource with key [url] and value [github repo url] 

### Result
The end result should look like the following:

![alt text](/img/artifactory-package-release-update-pl-rn.png)

# How to use git-package-release-update pipeline
You can use a pipeline to automate the process of extending, packaging and releasing your pipelines via a Git Release. The process is very similar to the section above. 

### Pre-reqs
- Fork this repository [devops-pipelines](https://github.com/ibm-garage-ref-storefront/devops-pipelines)

### Steps
1) Add your custom pipelines or modify an existing one 

    If you inspect `./pipelines/` you can create a new folder for each new pipeline you have and follow a similar structure as below.

    ```bash
    echo pwd
    ./devops-pipelines/pipelines
    ├── experimental
    │   ├── README.md
    │   ├── abc
    │   │   ├── bindings
    │   │   │   ├── abc-pl-pullrequest-binding.yaml
    │   │   │   └── abc-pl-push-binding.yaml
    │   │   ├── configmaps
    │   │   │   └── abc-pl-configmap.yaml
    │   │   ├── pipelines
    │   │   │   └── abc-pl.yaml
    │   │   ├── secrets
    │   │   │   └── abc-pl-secret.yaml
    │   │   ├── tasks
    │   │   │   └── abc-task.yaml
    │   │   └── template
    │   │       └── abc-pl-template.yaml
    │   └── manifest.yaml
    ├── stable
    │   ├── README.md
    │   ├── cde
    │   │   ├── bindings
    │   │   │   ├── cde-pl-pullrequest-binding.yaml
    │   │   │   └── cde-pl-push-binding.yaml
    │   │   ├── configmaps
    │   │   │   └── cde-pl-configmap.yaml
    │   │   ├── pipelines
    │   │   │   └── cde-pl.yaml
    │   │   ├── secrets
    │   │   │   └── cde-pl-secret.yaml
    │   │   ├── tasks
    │   │   │   └── cde-task.yaml
    │   │   └── template
    │   │       └── cde-pl-template.yaml
    │   └── manifest.yaml

    ```
    **Pipelines in `expermental` do not get built.**

2) Now drag and drop your pipelines and tasks to any of these folders, 
     

3) You must update the `configmap` and `secret` we provided for you. But first, create another repository such as `devops-server`. In this repo `devops-server` you will be hosting your pipelines as Git releases. Do not forget to create a README.md file.

    Navigate to `pipelines/stable/git-package-release-update/configmaps` and update the `pipeline-server-configmap.yaml`

    ```yaml
    apiVersion: v1
    kind: ConfigMap
    metadata:
    name: pipeline-server-configmap
    namespace: kabanero
    data:
        repo_org: your-github-username-or-org
        repo_name: your-github-repo-where-you-will-host-pipelines
        image_registry_publish: 'false'
        kabanero_pipeline_id: pipeline-manager
    ```

    Update the secret in `pipelines/stable/git-package-release-update/secrets/`

    ```yaml
    apiVersion: v1
    kind: Secret
    metadata:
        name: pipeline-server-git
        namespace: kabanero
        type: kubernetes.io/basic-auth
    data:
        password: your-git-token-encoded
        username: your-git-username-encoded
    ```
    
    Now run the following command to be able to retrieve resources for the `kabanero-pipeline` service account.

    ```bash
    oc adm policy add-cluster-role-to-user view system:serviceaccount:kabanero:kabanero-pipeline
    ```
    
4) Create web hook for the `devops-pipelines` repository you created on step 1.

5) Deploy your pipeline, tasks, event bindings and trigger templates by running the following command in the `devops-pipelines` repo you created on step 1:

    ```bash
    oc apply --recursive --filename pipelines/stable/git-package-release-update
    git add .
    git commit -m "adding new pipelines..."
    git push
    ```
### Result
Your output should be the following:
![](gifs/git-package-release-update-pl-run.gif)
<br>
If you go to the `devops-server` repo you created on step 2, you should see a new release with your zip files as shown below:
![](gifs/pipelines-server-release.gif)
<br>
Now inspect your Kabanero Custom Resource to ensure your `default-kabanero-pipelines.tar.gz` got added to the `pipelines` key value pair.
```bash
oc get kabanero -o yaml
```
```yaml
stacks:
    pipelines:
    - https:
        url: https://github.com/ibm-garage-ref-storefront/pipelines-server/releases/download/1.0/default-kabanero-pipelines.tar.gz
    id: pipeline-manager
    sha256: 8fe10018016e5059640b1a790afe2d6a1ff6c4f54bf3e7e4fa3fc0f82bb2207d
```
The pipelines that you added to the `devops-pipelines` repository should now be visible on the tekton dashboard as shown below:
![](img/deploy-storefront-ms-to-openshift.png)
<br>
![](img/git-package-release-pl.png)  

Now you can reuse these pipelines across your organization! If your cluster comes down you now have a backup of your pipelines.

# Create a tekton webhook 
### Pre-reqs

You need to create an access token on the tekton dashboard or cli in the kabanero namespace.
Earlier you created a github token on the github dashboard. You will need to get that token or generate another one and 
paste it below.
![](/gifs/access-token.gif)

Web hook Settings:

    Name: devops-demo-kabanero-pipelines
    Repistory-url: your forked repo url goes here
    Access Token: Token you generated previously 

Target Pipeline Settings
        
    Namespace: kabanero
    Pipeline: Choose artifactory-package-release-update-pl or git-package-release-update-pl
    Service Account: Pipeline
    Docker Registry: us.icr.io/project-name or docker.hub.io/projectname
        


