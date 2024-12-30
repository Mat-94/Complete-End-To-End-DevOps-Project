Seamless Deployment and Monitoring of a Zomato-Inspired Application using Jenkins, Docker, SonarQube, and Kubernetes on AWS

Tools & Services Used:
GitHub GitHub
Jenkins Jenkins
SonarQube SonarQube
Docker Docker
Kubernetes Kubernetes
Prometheus Prometheus
Grafana Grafana
ArgoCD ArgoCD
OWASP OWASP
Trivy Trivy
Project Stages:
Stage 1 - Deployment of App to Docker Container
Stage 2 - Deployment of App to K8S Cluster with Monitoring

Introduction: 
In today’s fast-paced software development landscape, automating deployment and ensuring seamless application monitoring are critical for maintaining efficiency and reliability. This project demonstrates a comprehensive DevOps CI/CD pipeline for deploying a Zomato-inspired web application using industry-standard tools and technologies.
The deployment workflow integrates Jenkins for continuous integration and delivery, Docker for containerization, SonarQube for static code analysis, and Kubernetes (via Amazon EKS) for scalable orchestration. Additionally, Prometheus and Grafana are configured for robust monitoring and alerting to ensure optimal performance and availability.
This project not only emphasizes deployment automation but also highlights best practices in resource provisioning, pipeline optimization, and infrastructure as c![Screenshot (199)](https://github.com/user-attachments/assets/48ad20ca-5cfe-4c23-b250-f81fd374182a)
ode (IaC) principles. It serves as an end-to-end reference for building scalable, reliable, and monitored web applications in a cloud-native environment.
1. Launch an Instance (Ubuntu, 24.04, t2.large, 30 GB)

2. Connect to the instance
3. ![Screenshot (197)](https://github.com/user-attachments/assets/9fba7723-6490-4cc7-92d7-0ca426e6891d)
![Screenshot (196)](https://github.com/user-attachments/assets/a2a83f3c-b960-40b4-975c-22d03d908740)

3. Update the packages
$ switch to root user ---> sudo su
$ sudo apt update -y
5. Install Jenkins on Ubuntu

6. Install Docker on Ubuntu
 7. Install Trivy on Ubuntu
 8. Install Docker Scout
 9. 9. Install SonarQube using Docker
![Screenshot (199)](https://github.com/user-attachments/assets/0957916f-bf97-4390-acec-1ef26488f3ca)
10. Installation of Plugins in Jenkins
Install below plugins:


11. SonarQube configuration in Jenkins


11.1. Tools Configuration in Jenkins
![Screenshot (200)](https://github.com/user-attachments/assets/5cc03414-9bd5-416c-be78-681dd0667faa)


11.2. Configuration of SonarQube Token in Jenkins
![Screenshot (206)](https://github.com/user-attachments/assets/9eaf6447-5d8f-49a5-b21a-ec01f2b7672b)

12. System Configuration in Jenkins

13. Create webhook in SonarQube
Create Pipeline Job
![Screenshot (210)](https://github.com/user-attachments/assets/238e316a-8e1d-4685-9e12-9c143e8342f4)

this is our application on the web
![Screenshot (211)](https://github.com/user-attachments/assets/9cbc8bc2-2efe-4321-a0ec-66057d905dfc)
the image below also shows our application pushed to docker hub
![Screenshot (206)](https://github.com/user-attachments/assets/2aa8cf39-b048-41a2-93fa-3df161d0d6f6)

MONITORING OF APPLICATION
------------------------------------------------------------
15. Launch VM (Name: Monitoring Server, Ubuntu 24.04, t2.large, Select the SG created in the Step 1, EBS: 30GB)
We will install Grafana, Prometheus, Node Exporter in the above instance and then we will monitor
--------------------------------------------------
15.1. Connect to 'Monitoring Server' VM
--------------------------------------------------

--------------------------------------------------
15.2. Installing Prometheus
![Screenshot (214)](https://github.com/user-attachments/assets/42f175ff-33da-4a22-85e2-bb2d7d245f07)

15.3. Installing Node Exporter 
--------------------------------------------------
cd 
You are in ~ path now

Create a system user for Node Exporter and download Node Exporter:
sudo useradd --system --no-create-home --shell /bin/false node_exporter
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.1/node_exporter-1.6.1.linux-amd64.tar.gz

Extract Node Exporter files, move the binary, and clean up:
tar -xvf node_exporter-1.6.1.linux-amd64.tar.gz
sudo mv node_exporter-1.6.1.linux-amd64/node_exporter /usr/local/bin/
rm -rf node_exporter*

Create a systemd unit configuration file for Node Exporter:
sudo vi /etc/systemd/system/node_exporter.service

Add the following content to the node_exporter.service file:
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter --collector.logind

[Install]
WantedBy=multi-user.target

Note: Replace --collector.logind with any additional flags as needed.

Enable and start Node Exporter:
sudo systemctl enable node_exporter
sudo systemctl start node_exporter

Verify the Node Exporter's status:
sudo systemctl status node_exporter
You can see "active (running)" in green colour
Press control+c to come out of the file

------------------------------------------------------------
15.4: Configure Prometheus Plugin Integration
------------------------------------------------------------
As of now we created Prometheus service, but we need to add a job in order to fetch the details by node exporter. So for that we need to create 2 jobs, one with 'node exporter' and the other with 'jenkins' as shown below;

Integrate Jenkins with Prometheus to monitor the CI/CD pipeline.

Prometheus Configuration:

To configure Prometheus to scrape metrics from Node Exporter and Jenkins, you need to modify the prometheus.yml file. 
The path of prometheus.yml is; cd /etc/prometheus/ ----> ls -l ----> You can see the "prometheus.yml" file ----> sudo vi prometheus.yml ----> You will see the content and also there is a default job called "Prometheus" Paste the below content at the end of the file;

  - job_name: 'node_exporter'
    static_configs:
      - targets: ['<MonitoringVMip>:9100']

  - job_name: 'jenkins'
    metrics_path: '/prometheus'
    static_configs:
      - targets: ['<your-jenkins-ip>:<your-jenkins-port>']

 In the above, replace <your-jenkins-ip> and <your-jenkins-port> with the appropriate IPs ----> esc ----> :wq

Check the validity of the configuration file:
promtool check config /etc/prometheus/prometheus.yml

You should see "SUCCESS" when you run the above command, it means every configuration made so far is good.

Reload the Prometheus configuration without restarting:
curl -X POST http://localhost:9090/-/reload

Access Prometheus in browser (if already opened, just reload the page):
http://<your-prometheus-ip>:9090/targets

Open Port number 9100 for Monitoring VM 

You should now see "Jenkins (1/1 up)" "node exporter (1/1 up)" and "prometheus (1/1 up)" in the prometheus browser.
Click on "showmore" next to "jenkins." You will see a link. Open the link in new tab, to see the metrics that are getting scraped
![Screenshot (216)](https://github.com/user-attachments/assets/0220a61c-a72e-4dbf-a2b9-a10125836ad6)

--------------------------------
 Install Grafana
------------------------------------------------------------
You are currently in /etc/Prometheus path.

Install Grafana on Monitoring Server;

Step 1: Install Dependencies:
First, ensure that all necessary dependencies are installed:
sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common

Step 2: Add the GPG Key:
cd ---> You are now in ~ path
Add the GPG key for Grafana:
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -

You should see OK when executed the above command.

Step 3: Add Grafana Repository:
Add the repository for Grafana stable releases:
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list

Step 4: Update and Install Grafana:
Update the package list and install Grafana:
sudo apt-get update
sudo apt-get -y install grafana

Step 5: Enable and Start Grafana Service:
To automatically start Grafana after a reboot, enable the service:
sudo systemctl enable grafana-server

Start Grafana:
sudo systemctl start grafana-server

Step 6: Check Grafana Status:
Verify the status of the Grafana service to ensure it's running correctly:
sudo systemctl status grafana-server

You should see "Active (running)" in green colour
Press control+c to come out

Step 7: Access Grafana Web Interface:
The default port for Grafana is 3000
http://<monitoring-server-ip>:3000
![Screenshot (218)](https://github.com/user-attachments/assets/480ebd2c-3ffd-4c08-a035-320b616204c0)
You will see the Grafana dashboard.
![Screenshot (220)](https://github.com/user-attachments/assets/aaca026e-9fff-4dd5-b1fd-4e0918a9d0fc)


The first thing that we have to do in Grafana is to add the data source
Lets add the data source;


Click on Dashboards in the left pane, you can see both the dashboards you have just added.

![Screenshot (221)](https://github.com/user-attachments/assets/e32de29e-027a-479b-9d1f-b90ce445e9aa)

To verify the cluster creation ---> Goto Cloud Formation service in AWS ----> You should see a stack got created with the name "kastrocluster". Make sure in the vs code editor the cluster will get created. As said earlier it will take atleast 20 minutes.
Once the cluster is ready, you will see "EKS Cluster "kastrocluster" in "us-east-1" region is ready" in vs code editor. wait till you see this.
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
# Get List of clusters
eksctl get cluster
Step 02: Create & Associate IAM OIDC Provider for our EKS Cluster
To enable and use AWS IAM roles for Kubernetes service accounts on our EKS cluster, we must create & associate OIDC identity provider.
To do so using eksctl we can use the below commands.

# Template
eksctl utils associate-iam-oidc-provider \
    --region region-code \
    --cluster <cluster-name> \
    --approve

Step 03: Create Node Group with additional Add-Ons in Public Subnets
These add-ons will create the respective IAM policies for us automatically within our Node Group role.
Step 05: Verify Cluster & Nodes
Goto EKS Service in AWS and cLet us deploy the same application in the EKS cluster also
heck for the cluster creation

15.6: Argo CD installation
------------------------------------------------------------

Inorder to monitor k8s with Prometheus, we need to install ArgoCD. Lets do that
Execute the below commands in vs code editor

kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/v2.4.7/manifests/install.yaml

wait for sometime till the namespace gets created.
The above command will create a namespace with "argocd" name

By default the argo CD server is not publicly exposed, so we need to expose it publicly. To do that, execute the below command;
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

(OR) Command Prompt Execution
kubectl patch svc argocd-server -n argocd -p "{\"spec\": {\"type\": \"LoadBalancer\"}}"

After successful execution you should see "patched"

To see the namespace got created or not ----> kubectl get ns ----> you will see argocd namespace
To see the pods available in the argocd namespace ----> kubectl get pods -n argocd ----> you will see the pods

Wait for 5 minutes for the load balancer creation. Once the loadbalancer is created, we will get the load balancer url.

Meanwhile execute the below commands in vs code editor

------------------------------------------------------------
15.7: Monitor Kubernetes with Prometheus
------------------------------------------------------------
Used to monitor Kubernetes cluster.
Additionally, you'll install the node exporter using Helm to collect metrics from your cluster nodes.

Install Node Exporter using Helm
To begin monitoring your Kubernetes cluster, you'll install the Prometheus Node Exporter. This component allows you to collect system-level metrics from your cluster nodes. Here are the steps to install the Node Exporter using Helm:

Add the Prometheus Community Helm repository:
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

Create a Kubernetes namespace for the Node Exporter:
kubectl create namespace prometheus-node-exporter

Install the Node Exporter using Helm:
helm install prometheus-node-exporter prometheus-community/prometheus-node-exporter --namespace prometheus-node-exporter

Lets continue with load balancer thing of previous step; execute the below in VS code editor
export ARGOCD_SERVER=`kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].hostname'`

Execute the below command in powershell, if the command doesn't get executed in VS Code Editor
$env:ARGOCD_SERVER = $(kubectl get svc argocd-server -n argocd -o json | jq --raw-output '.status.loadBalancer.ingress[0].hostname')


(Ref URL: https://archive.eksworkshop.com/intermediate/290_argocd/configure/)

To get the loadbalancer url;
echo $ARGOCD_SERVER

Execute the below command in powershell, if the command doesn't get executed in VS Code Editor
echo $env:ARGOCD_SERVER

You will see the load balancer url, copy it and paste in browser. You will see the ArgoCD Homepage.
Username is "admin"
To get the password, execute the below command in vs code editor;
export ARGO_PWD=`kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d`

Execute the below command in powershell, if the command doesn't get executed in VS Code Editor
$env:ARGO_PWD = (kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | % { [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($_)) })

To see the password;
echo $ARGO_PWD

Execute the below command in powershell, if the command doesn't get executed in VS Code Editor
echo $env:ARGO_PWD

You will see the password. copy and paste it in the argo cd homepage --->login

<Follow the process as explained in the video>

Note: In the repo, in Kubernetes folder, in the deployment.yml file, in the containers section change the dockerhub username

Add a Job to Scrape Metrics on nodeip:9001/metrics in prometheus.yml:

Update your Prometheus configuration (prometheus.yml) to add a new job for scraping metrics from nodeip:9001/metrics. You can do this by adding the following configuration to your prometheus.yml file:
Go to the monitoring server tab in Moba and execute the below commands;
sudo vi /etc/prometheus/prometheus.yml ----> Paste the below commands at the bottom of screen ----> 
  - job_name: 'k8s'
    metrics_path: '/metrics'
    static_configs:
      - targets: ['nodeIP:9100']

In the above, to get the "nodeIP", goto EKS in AWS ----> Click on EKS Cluster ----> "Compute" tab ----> Nodes ----> Click on any one node ----> Click on the "instance id" ----> Copy the public ip ----> Paste in the above script

The static_configs section specifies the targets to scrape metrics from, and in this case, it's set to nodeip:9001.

----> esc ----> :wq ----> promtool check config /etc/prometheus/prometheus.yml ----> You should see "Success" ----> Check the validity of the configuration file ----> promtool check config /etc/prometheus/prometheus.yml ----> curl -X POST http://localhost:9090/-/reload

Goto Prometheus and reload. Goto ArgoCD and reload to see whether the pipeline is done or not

Copy the public ip of "nodeIP" which we have done exactly 4 steps above this line ---> Goto browser and paste it:30001 ----> Make sure to open the port 30001 for the "nodeIP:" VM ----> You will see the application

Note: If you see error in Prometheus under "k8s", open port number 9100 for the EC2 instances which were created as part of EKS cluster i.e nodes



