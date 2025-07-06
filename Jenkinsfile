pipeline{
     agent any
     
     tools{
         jdk 'jdk17'
         nodejs 'node16'
     }
     environment {
         SCANNER_HOME=tool 'sonarqube-scanner'
     }
     
     stages {
         stage('Clean Workspace'){
             steps{
                 cleanWs()
             }
         }
         stage('Checkout from Git'){
             steps{
                 git branch: 'main', url: 'https://github.com/itsrizkan/Clone-App-CICD.git'
             }
         }
         stage("Sonarqube Analysis "){
             steps{
                 withSonarQubeEnv('SonarQube-Server') {
                     sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Clone-App-CICD \
                     -Dsonar.projectKey=Clone-App-CICD '''
                 }
             }
         }
         stage("Quality Gate"){
            steps {
                 script {
                     waitForQualityGate abortPipeline: false, credentialsId: 'SonarQube-Token' 
                 }
             } 
         }
         stage('Install Dependencies') {
             steps {
                 sh "npm install"
             }
         }
         stage('TRIVY FS SCAN') {
             steps {
                 sh "trivy fs . > trivyfs.txt"
             }
         }
          stage("Docker Build & Push"){
             steps{
                 script{
                    withDockerRegistry(credentialsId: 'dockerhub', toolName: 'docker'){   
                        sh "docker build -t clone-app ."
                        sh "docker tag clone-app itsrizkan/clone-app:latest "
                        sh "docker push itsrizkan/clone-app:latest "
                     }
                 }
             }
         }
         stage("TRIVY"){
             steps{
                 sh "trivy image itsrizkan/clone-app:latest > trivyimage.txt" 
             }
         }
          stage('Deploy to Kubernets'){
             steps{
                 script{
                     dir('Kubernetes') {
                         withkubeconfig(caCertificate: '', clusterName: '', contextName: '', credentialsId: 'kubernetes', namespace: '', restrictKubeConfigAccess: false, serverUrl: '') {
                         sh 'kubectl delete --all pods'
                         sh 'kubectl apply -f deployment.yml'
                         sh 'kubectl apply -f service.yml'
                         }   
                     }
                 }
             }
         }



     }
 }