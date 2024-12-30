Seamless Deployment and Monitoring of a Zomato-Inspired Application using Jenkins, Docker, SonarQube, and Kubernetes on AWS
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



