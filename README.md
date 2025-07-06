Building a CICD with Jenkins using Terraform, AWS, SonarQube, Docker, EKS, and Trivy.

1. Install Terraform (local)
2. Insall AWS CLI (local) & Create IAM user for Terraform resource creation with relevant policies & get the CLI access key & configure local terminal
3. Create local repo and open with vscode 
4. Create main.tf file to create JenkinsSonarQube EC2
5. Create provider.tf terraform file to configure the AWS details
6. Create the sh template to define for the EC2 (attached to main.tf )
7. Using local terminal (vscode terminal) build all the Terraform files (terraform init, terraform plan, terraform apply -auto-approve)
8. Configure Jenkins (connect to th EC2 and get the intial pw, hit the URL (public ip:8080) and intall suggested plugins, setup new pw)
9. Install following additional plugins and restart (Eclipse Temurin Installer, SonarQube Scanner, Sonar Quality Gates, Quality Gates, NodeJS, Docker, Docker API, Docker Commons, Docker Pipeline, docker-build-step)
10. Intall Jenkins tools (add nodejs > name:node16 > insalll from:nodejs.org ) (add jdk > name:jdk17 > install from:adoptium.net > version: jdk-17.0.8.1+1) & apply & save (add docker > name:docker > install auto > download from: docker.com) (add sonaqube > name:sonarqube-scanner > install auto > install from:maven central ) apply & save
11. Configure SonarQube & intergrate with jenkins (EC2PublicIP:9000), generate tocken for Jenkins from SonarQube & add that credentials to Jenkins > (manage Jenkins>system>SonarQube servers>add SonarQube>name:Clone-App-CICD>url:http://EC2PrivateIP:9000>select created tocken) apply & save
12. Create SonarQube quality gate ( name:SonarQube-QualityGate) && create webhook (administration>config>webhooks>create>name:jenkins>url:http://EC2PrivateIP:8080/soarqube-webhook)
13. Create Jenkins pipeline (new item>pipeline>name:Clone-App-CICD>check on discard old builds>max:2>build trigger:nownow>write the script:Jenkinsfile) 
14. Configure DockerHub & intergrate with Jenkins (login to DockerHub> create tocken> add to Jenkins)
15. Configure AWS EKS :> Install kubectl on EC2 (Jenkins Server) > Install AWS CLI on EC2 (Jenkins Server) > Install  eksctl on EC2 (Jenkins Server) > create iam role and attach to EC2 (IAM>Roles>AWS service>EC2>AdministratorAccess>roleName:eksctl_role>EC2>action>security>modify>select role>update)

  15.1 Install kubectl on Jenkins Server
   sudo apt update
   sudo apt install curl
   curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   kubectl version --client

  15.2 Install AWS cli
   curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
   sudo apt install unzip
   unzip awscliv2.zip
   sudo ./aws/install
   aws --version

  15.3 Install  eksctl
   curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp                    #package created on /tmp directory
   cd /tmp                         #switch to /tmp
   sudo mv /tmp/eksctl /bin        #move the package to /bin (since its an executable file)
   eksctl version

16. Setup Kubernetes using eksctl

  16.1 Setup Kubernetes using eksctl
   eksctl create cluster --name virtualtechbox-cluster \
   --region ap-south-1 \
   --node-type t2.small \
   --nodes 3 \

  16.2 Verify Cluster with below command
   $ kubectl get nodes
   $ kubectl get svc

17. Copy kube config file to local (/home/ubuntu/.kube) > open the copied file & copy the content & save as secret.txt file on local         #this file will be used to intergrate EKS with jenkins
18. Install additional jenkins plugins (Kubernetes, Kubernetes Credentials, Kubernetes Client API, Kubernetes CLI)
19. Add the Kubernetes credentials to Jenkins (kind:secret file, id:kubernetes, choose file: secret.txt created at step-17)
20. Set trigger > select Pipeline>Configuration>tick github project>give project url>tick on GitHub hook trigger for GITScm polling
21. Go to github repo settings>webhooks>add payload url(Jenkins url> http://EC2PublicIP:8080/github-webhook>add webhook)
22. Verify by doing a new commit and pushin to reomete repo, this should trigger the pipeline > also verify app changes by getting EKs pods service URL ($ kubectl get svc)
