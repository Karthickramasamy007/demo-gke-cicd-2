# AUTHOR
- Karthick ramsamy
- date started : 19/12/2022
- status : on going..
- date completed :


# CONTENET
* This project contains:

    - gcloud configuration, 
    - gcp project creation, 
    - link billing to project, 
    - enabeling api for gke, 
    - creating gke 'dev' cluster with one node,

    - deploying jenkins 'dev' cluster with persistant volume (** skipping PV for now)
    - configure jenkins node to run build jobs on 'dev'cluster

    - creating multiple microservices applications for demo

    - configure jenkins for cicd pipeline to deploy multiple microservices to 'dev' and 'sit' cluster.

    - creating node port-type service and check it's access
    - creating load-balancer type service check it's access
    - create cluster-ip type service and check it's access
    - create a cluster network and check it's access *** STUDY FROM YT

    - deploy fluentd as a demon set in 'dev' and 'sit'
    - collect application logs using fluentd
    - collect kubernetes logs using fluentd

    - deploy prometheus in 'dev' and 'sit' cluster
    - configure prometheus to monitor dev and sit cluster
    - configure prometheus to monitor applications in dev and sit cluster

    * FEATURE UPDATES
    - create gke dev and sit cluster using terraform
    - start exploring Ansible into this project



--- STEPS TO FOLLOW ---

# GCLOUD CONFIGURATION
* Make sure you have gcloud cli is installed in your local machine.
* Setup gcloud account and gcloud cli configuration
* gcloud init
* using the above command we have created gcloud config called  'demo-dev-gcloud-configuration'
* Now you can make sure your active gcloud account by (* is active): gcloud auth list
* You can also check properties of our created active gcloud config 'demo-dev-gcloud-configuration' by : gcloud config list

# CREATE PROJECT
* create project : gcloud projects create dev-demo-proj-id --name="demo-dev-project" --labels=type=dev
* you can check it by : gcloud projects list --sort-by=projectId --limit=5

# BILLING
* we need to create and link billing account with our new project.
* created billing account via UI
* to list all available billing accoounts for current active user: gcloud beta billing accounts list
* to link project to billing : gcloud beta billing projects link <project-id> --billing-account <billing-account-id>

# ENABLE API'S
* enable api for gke cluster: gcloud services enable container.googleapis.com

# CREATE GKE CLUSTER
* gcloud container clusters create demo-dev-gke-cluster --region us-west1 --node-locations us-west1-c --num-nodes 1 --cluster-version latest
* describe cluster: gcloud container clusters describe demo-dev-gke-cluster --region us-west1
* your local kube config and current-context will automatically get updated with this latest gke cluster details. Now you can run kubectl commands from your local machine. You can check the current context by: kubectl config current-context 
* Or you can view full kube config by : kubectl config view
* you can get the cluster ip from kube config file using above command or from .kube file in user's home path.

# SET K8'S CONTEXT TO DEV CLUSTER
* By default kube config will set to use our latest dev cluster. you can check this by : kube config current-context
* if you want to set it :  kubectl config use-context <DEV-CLUSTER>

# DEPLOY AND RUN JENKINS IN DEV K8'S CLUSTER
* cd infra
* kubectl create ns jenins
* kubectl apply -f . -n jenkins
* The following step is only needed if you set svc 'type' as 'NodePort'. As we set to 'LoadBalancer' type we don't need to port forward. we can skip the following step.
* kubectl port-forward svc/jenkins 8080:8080 50000:50000 -n jenkins
* for NodePort user node ip with jenkins port to access jenkins service. For LoadBalancer user load balancer ip with jenkins port. You can get these port by below command.
* kubectl get svc -n jenkins
* Now, get admin password from jenkins pod log:
* kubectl get pods -n jenkins
* kubectl logs <pod-name> -n jenkins
* access jenkins service from browser : http://localhost:8080
* create user account and install suggested plugins.

# SET JENKINS NODE CONFIGURATION TO RUN AND DEPLOY APP ON 'DEV' K8'S
* go to manage jenkins --> manage plug-in : install plug-in called 'kubernetes' to manage jenkis node to run on kubernets as a pod to execute build steps
    and jenkins to deploy directly to variouse k8's clusters including k8's running on public cloud like gcp, aws and also in same cluster where jenkins is running.
* go to manage jenkins, mange nodes and clouds, configure clouds, add new clouds, and click Kubernetes.
    Now, Give name as 'kind_dev_cluster' insted of kubernetes. (This name must match with agent cloud name Jekinsfile. otherwise jenkins will take 'kubernets' as default). 
    Now put kuberenets url where you want the jenkins node to run it's build jobs. In this case, dev cluster. You can get the cluster ip and port from kube config file. Get apiServerAddress: "172.20.128.1" and 
    apiServerPort: 58350 from the kind cluster config for dev cluster. eg https://172.20.128.1:58350
* Now click the test connection to test the connection with K8's. 
    If you put remote k8's you need to pass the certificate key to get authenticated or need to pass it's config file that containes cluster url, certificate and user with proper service account set. If you use config file we no need to pass kubernetes url in the text box as it will be in the config file itself.
* Put name space as 'jenkins'
* jenkins url : http://jenkins:8080 . It also can be public dns. but it is not safer as Jenkkins is not good with handling sercurity and stuffs.
* Set jenkins tunnel url no need to put http infront. To get the ip address of this service: kubectl describe svc/jenkins -n jenkins
* Get the jenkins ip for port 50000. eg : 10.244.0.5:50000 
* We don't need to set pod template and container template here in this plug-in as we create it with pipeline script.

# SET APP TO DEPLOY ON 'SIT' K8'S CLUSTER AS WELL
* go to manage jenkins, mange nodes and clouds, configure clouds, add new clouds, and click Kubernetes.
    Now, Give name as 'kind_sit_cluster' insted of kubernetes. (This name must match with agent cloud name Jekinsfile. otherwise jenkins will take 'kubernets' as default). 
    Now put kuberenets url where you want the jenkins node to run it's build jobs. In this case, dev cluster. You can get the cluster ip and port from kube config file. Get apiServerAddress: "172.20.128.1" and 
    apiServerPort: 58350 from the kind cluster config for dev cluster. eg https://172.20.128.1:58350
* Here, we don't need to give jenkins url and tunnel url as we are only deploying app to this 'SIT' cluster. We as not tunning jenkins node to build jobs here. Jenkins running in
    'DEV' cluster will deploy app directly to this 'SIT' cluster through cluster api and service account.

# SET DOCKER CREDENTINALS AS K8'S SECRET
* kubectl create secret docker-registry regcred --docker-server=https://index.docker.io/v1/ --docker-username=<docker-username> --docker-password=<docker-password> --docker-email=<docker-email> -n jenkins
* Here we have cerated secret called regcred which has login details to docker hub to pull and push the images from.
* We have mounted this k8's secret path wich has docker login details to the 'Kaniko' container as this uses this secret to push the images to docker that it built.


# CREATE PIPELINE PROJECT
* Now create a 'pipeline' projet with name that you want by clicking new items. Here we call it "demo".
* Now go to pipeline section and select 'pipeline script from scm' and seletec 'Git' as scm.
* Now put git repository URL without .git at the end.
* Now put path to 'jenkisfile' that containes pipline script. Here we have to give pipeline/jenkinsfile
* Now mention the branch as */master. Here our project in 'master' branch. Whenever, we merge to the master brach the pipeline will get triggered.
* Now click save and start building the pipeline.


# CONFIGURE GIT REPO AND PUSH CODE:
* Get into the folder that you want to push into git. Here 'cicd-demo-1'
* Also, create a repo called 'demo-gke-cicd-2' in git hub using UI.
git init
git add README.md
git commit -m "first commit"
git branch -M main
git remote add origin https://github.com/Karthickramasamy007/demo-gke-cicd-2.git
git push -u origin main


