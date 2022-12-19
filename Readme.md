# AUTHOR
- Karthick ramsamy
- date started : 19/12/2022
- date completed :


# CONTENET




# SET K8'S CONTEXT TO DEV CLUSTER
* kubectl config use-context <DEV-CLUSTER>

    # DEPLOY AND RUN JENKINS IN DEV K8'S CLUSTER
    * cd infra
    * kubectl create ns jenins
    * kubectl apply -f . -n jenkins
    * kubectl port-forward svc/jenkins 8080:8080 50000:50000 -n jenkins
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


