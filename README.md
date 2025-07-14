# Building a CI/CD with Jenkins using Terraform, AWS, SonarQube, Docker, EKS, and Trivy

## Steps

1. **Install Terraform (local)**

2. **Install AWS CLI (local)**  
   - Create IAM user for Terraform resource creation with relevant policies  
   - Get the CLI access key  
   - Configure local terminal

3. **Create local repo and open with VSCode**

4. **Create `main.tf` file** to create JenkinsSonarQube EC2

5. **Create `provider.tf`** Terraform file to configure the AWS details

6. **Create the `.sh` template** to define EC2 initialization (attach to `main.tf`)

7. **Using VSCode terminal**, build all the Terraform files:

   ```bash
   terraform init
   terraform plan
   terraform apply -auto-approve
   ```

8. **Configure Jenkins**  
   - Connect to the EC2  
   - Get the initial password  
   - Access `http://<Public-IP>:8080`  
   - Install suggested plugins  
   - Set new password

9. **Install additional Jenkins plugins and restart:**  
   - Eclipse Temurin Installer  
   - SonarQube Scanner  
   - Sonar Quality Gates  
   - Quality Gates  
   - NodeJS  
   - Docker  
   - Docker API  
   - Docker Commons  
   - Docker Pipeline  
   - docker-build-step

10. **Install Jenkins tools**  
    - Add NodeJS  
      - Name: `node16`  
      - Install from: `nodejs.org`  
    - Add JDK  
      - Name: `jdk17`  
      - Install from: `adoptium.net`  
      - Version: `jdk-17.0.8.1+1`  
    - Add Docker  
      - Name: `docker`  
      - Install automatically from: `docker.com`  
    - Add SonarQube  
      - Name: `sonarqube-scanner`  
      - Install automatically from: `Maven Central`  
    - Apply & Save

11. **Configure SonarQube & integrate with Jenkins**  
    - Access SonarQube: `http://<EC2-Public-IP>:9000`  
    - Generate token for Jenkins from SonarQube  
    - In Jenkins:  
      - Go to `Manage Jenkins > System > SonarQube Servers`  
      - Add SonarQube  
      - Name: `Clone-App-CICD`  
      - URL: `http://<EC2-Private-IP>:9000`  
      - Select the generated token  
      - Apply & Save

12. **Create SonarQube Quality Gate and Webhook**  
    - Quality Gate Name: `SonarQube-QualityGate`  
    - Webhook:  
      - Go to `Administration > Configuration > Webhooks`  
      - Name: `jenkins`  
      - URL: `http://<EC2-Private-IP>:8080/soarqube-webhook`

13. **Create Jenkins Pipeline**  
    - New Item > Pipeline  
    - Name: `Clone-App-CICD`  
    - Check: Discard old builds (max: 2)  
    - Build trigger: Now  
    - Pipeline script: `Jenkinsfile`

14. **Configure DockerHub & integrate with Jenkins**  
    - Log in to DockerHub  
    - Create token  
    - Add token to Jenkins credentials

15. **Configure AWS EKS (on EC2 Jenkins Server)**

    **15.1 Install `kubectl`**

    ```bash
    sudo apt update
    sudo apt install curl
    curl -LO https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl
    sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
    kubectl version --client
    ```

    **15.2 Install AWS CLI**

    ```bash
    curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
    sudo apt install unzip
    unzip awscliv2.zip
    sudo ./aws/install
    aws --version
    ```

    **15.3 Install `eksctl`**

    ```bash
    curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp
    cd /tmp
    sudo mv /tmp/eksctl /bin
    eksctl version
    ```

    - Create IAM Role and attach to EC2:  
      - IAM > Roles > AWS Service > EC2  
      - Attach `AdministratorAccess`  
      - Role Name: `eksctl_role`  
      - EC2 > Actions > Security > Modify IAM Role > Select role > Update

16. **Setup Kubernetes using eksctl**

    **16.1 Create Cluster**

    ```bash
    eksctl create cluster --name virtualtechbox-cluster \
      --region ap-south-1 \
      --node-type t2.small \
      --nodes 3
    ```

    **16.2 Verify Cluster**

    ```bash
    kubectl get nodes
    kubectl get svc
    ```

17. **Copy Kube Config File to Local**  
    - Path: `/home/ubuntu/.kube/config`  
    - Open the file, copy content, and save as `secret.txt` locally  
    - This file will be used to integrate EKS with Jenkins

18. **Install Additional Jenkins Plugins**  
    - Kubernetes  
    - Kubernetes Credentials  
    - Kubernetes Client API  
    - Kubernetes CLI

19. **Add Kubernetes Credentials to Jenkins**  
    - Kind: Secret file  
    - ID: `kubernetes`  
    - Choose file: `secret.txt` (from Step 17)

20. **Set Trigger for GitHub Integration**  
    - Jenkins Project > Configuration  
    - Tick: "GitHub project"  
    - Enter project URL  
    - Tick: "GitHub hook trigger for GITScm polling"

21. **Configure GitHub Webhook**  
    - Go to GitHub Repo > Settings > Webhooks  
    - Add payload URL:  
      ```
      http://<EC2-Public-IP>:8080/github-webhook/
      ```

22. **Verify Pipeline Trigger**  
    - Make a commit & push to remote repo  
    - Check if Jenkins pipeline is triggered  
    - Use EKS service URL to verify app:

    ```bash
    kubectl get svc
    ```
