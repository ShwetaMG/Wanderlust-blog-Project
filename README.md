# Wanderlust - A travel blog app🌍✈️

WanderLust is a simple MERN travel blog website ✈

![Preview Image](https://github.com/krishnaacharyaa/wanderlust/assets/116620586/17ba9da6-225f-481d-87c0-5d5a010a9538)
#

# Steps for Implementation

### Follow the below steps to deploy three tier MERN stack application on EKS cluster.
#

> [!Note]
> This assumes you've already completed all the tool installation and setup steps in [README-Installation.md](./README-Installation.md) (Jenkins Master/Worker, EKS cluster, ArgoCD, SonarQube, Trivy, Email notification, Helm monitoring). This document covers configuring the pipeline itself and deploying the application.

### <mark>Project Deployment Flow:</mark>
<img src="https://github.com/DevMadhup/Wanderlust-Mega-Project/blob/main/Assets/DevSecOps%2BGitOps.gif" />

#

### How pipeline will look after deployment:
- <b>CI pipeline to build and push</b>

![CI Pipeline](Images/CI-Pipeline.png)

- <b>CD pipeline to update application version</b>

![CD Pipeline](Images/CD-Pipeline.png)

- <b>ArgoCD application for deployment on EKS</b>

![ArgoCD Deployment](Images/argo-app-connected.png)

#
## Steps to implement the project:
- <b>Go to Jenkins Master and click on <mark> Manage Jenkins --> Plugins --> Available plugins</mark> install the below plugins:</b>
  - OWASP
  - SonarQube Scanner
  - Docker
  - Pipeline: Stage View
#
- <b id="Sonar">After OWASP plugin is installed, Now move to <mark>Manage jenkins --> Tools</mark> (Jenkins master)</b>

![Manage Jenkins Tools](Images/OWASP-dependency.png)
#
- <b>Login to SonarQube server and create the credentials for jenkins to integrate with SonarQube</b>
  - Navigate to <mark>Administration --> Security --> Users --> Token</mark>

  ![SonarQube Token Step 1](Images/sonar-qube-token-crea-1.png)
  ![SonarQube Token Step 2](Images/sonar-token-crea-2.png)

#
- <b>Now, go to <mark> Manage Jenkins --> credentials</mark> and add Sonarqube credentials:</b>

![SonarQube Credentials in Jenkins](Images/sonarqube-credentials.png)
#
- <b>Go to <mark> Manage Jenkins --> Tools</mark> and search for SonarQube Scanner installations:</b>

![SonarQube Scanner Tool Config](Images/Sonarqube-installation.png)
#
- <b> Go to <mark> Manage Jenkins --> credentials</mark> and add Github credentials to push updated code from the pipeline:</b>

![GitHub Credentials in Jenkins](Images/github-credentials.png)

> [!Note]
> While adding github credentials add Personal Access Token in the password field.
#
- <b>Go to <mark> Manage Jenkins --> System</mark> and search for SonarQube installations:</b>

![SonarQube Installation in System Config](Images/Sonarqube-installation.png)
#
- <b>Now again, Go to <mark> Manage Jenkins --> System</mark> and search for Global Trusted Pipeline Libraries:</b>

![Global Trusted Pipeline Libraries 1](Images/global-trusted-pipeline.png)
![Global Trusted Pipeline Libraries 2](Images/global-trusted-pipeline-2.png)
#
- <b>Login to SonarQube server, go to <mark>Administration --> Webhook</mark> and click on create </b>

![SonarQube Webhook 1](Images/sonarqube-webhook.png)
![SonarQube Webhook 2](Images/sonar-qube-webhook-1.png)
#
- <b>Now, go to github repository and under <mark>Automations</mark> directory update the <mark>instance-id</mark> field on both the <mark>updatefrontendnew.sh updatebackendnew.sh</mark> with the k8s worker's instance id</b>

![Update Instance ID in Automation Scripts](Images/Autofiles.sh-modification.png)
#
- <b>Navigate to <mark> Manage Jenkins --> credentials</mark> and add credentials for docker login to push docker image:</b>

![Docker Login Credentials](Images/dockerhub-credentials.png)
#
- <b>Create a <mark>Wanderlust-CI</mark> pipeline</b>

![Wanderlust CI Pipeline](Images/CI-Creation.png)

#
- <b>Create one more pipeline <mark>Wanderlust-CD</mark></b>

![Wanderlust CD Pipeline 1](Images/CD-creation.png)
![Wanderlust CD Pipeline 2](Images/CI-script.png)
![Wanderlust CD Pipeline 3](Images/CD-script.png)
#
- <b>Provide permission to docker socket so that docker build and push command do not fail (Jenkins Worker)</b>
```bash
chmod 777 /var/run/docker.sock
```
#
- <b> Go to Master Machine and add our own eks cluster to argocd for application deployment using cli</b>
  - <b>Login to argoCD from CLI</b>
  ```bash
   argocd login 52.66.65.178:31071 --username admin
  ```
> [!Tip]
> 52.66.65.178:31071--> This should be your argocd url

  ![ArgoCD CLI Login](Images/argo-terminal-login.png)

  - <b>Check how many clusters are available in argocd </b>
  ```bash
  argocd cluster list
  ```
  ![ArgoCD Cluster List](Images/cluster-list.png)
  - <b>Get your cluster name</b>
  ```bash
  kubectl config get-contexts
  ```
  ![Kubectl Contexts](Images/config-get-contex.png)
  - <b>Add your cluster to argocd</b>
  ```bash
  argocd cluster add shweta-admin@wanderlust.ap-south-1.eksctl.io  --name wanderlust-eks-cluster
  ```
  > [!Tip]
  > shweta-admin@wanderlust.ap-south-1.eksctl.io --> This should be your EKS Cluster Name.

  ![ArgoCD Cluster Add](Images/add-cluster-argocd.png)
  - <b> Once your cluster is added to argocd, go to argocd console <mark>Settings --> Clusters</mark> and verify it</b>

  ![ArgoCD Cluster Verify](Images/cluster-in-argo.png)
#
- <b>Go to <mark>Settings --> Repositories</mark> and click on <mark>Connect repo</mark> </b>

![ArgoCD Connect Repo 1](Images/argo-conn1.png)
![ArgoCD Connect Repo 2](Images/argo-conn2.png)
![ArgoCD Connect Repo 3](Images/argo-conn3.png)

> [!Note]
> Connection should be successful

- <b>Now, go to <mark>Applications</mark> and click on <mark>New App</mark></b>

![ArgoCD New Application](Images/argo-app-setup.png)

> [!Important]
> Make sure to click on the <mark>Auto-Create Namespace</mark> option while creating argocd application

![ArgoCD Auto Create Namespace 1](Images/argo-Auto-crea-namespace.png)
![ArgoCD Auto Create Namespace 2](Images/argo-source-conn.png)
![ArgoCD Auto Create Namespace 2](Images/argo-dest-conn.png)


- <b>Congratulations, your application is deployed on AWS EKS Cluster</b>

![Application Deployed 1](Images/argo-app-connected.png)
![Application Deployed 2](Images/ArgoCd-dashboard.png)

- <b>Open port 31000 and 31100 on worker node and Access it on browser using public IP address</b>
```bash
<worker-public-ip>:31000
```
![Application Running 1](Images/ec2-node-port.png)
![Application Running 2](Images/app-on-browser.png)
![Application Running 3](Images/application.png)

- <b>Email Notification</b>

![Email Notification Received](Images/Mail-notification.png)

#

- Now, view the Dashboard in Grafana
  
![Grafana Dashboard 2](Images/grafana-5.png) 
![Grafana Dashboard 1](Images/grafana-4.png)
![Grafana Dashboard 3](Images/grafana-6.png)

#
## Clean Up
- <b id="Clean">Delete eks cluster</b>
```bash
eksctl delete cluster --name=wanderlust --region=us-west-1
```

#

That's it! Your Wanderlust MERN application is now deployed on AWS EKS with a full DevSecOps pipeline: GitHub → Jenkins CI (OWASP + SonarQube + Trivy) → Docker → Jenkins CD → ArgoCD → EKS, with Prometheus/Grafana monitoring and email notifications.

For setup and installation of all the underlying tools, refer back to [README-Installation.md](./README-Installation.md).


> [!Note]
> completed this jenkin deployment by referring ShubamLonde youtube "Jenkin oneshort video" please go through yotube channel video for any doubts and clarifications.
