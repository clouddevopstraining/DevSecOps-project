pipeline {
    agent any
    
    stages {
        stage('Checkout git') {
            steps {
               git branch: 'main', url: 'https://github.com/clouddevopstraining/DevSecOps-project.git'
            }
        }
        
        stage ('Build & JUnit Test') {
            steps {
                sh 'mvn install' 
            }
            post {
               success {
                    junit 'target/surefire-reports/**/*.xml'
                }   
            }
        }
        stage('SonarQube Analysis'){
            steps{
                withSonarQubeEnv('SonarCloud') {
                        sh 'mvn clean verify sonar:sonar \
                        -Dsonar.projectKey=DevSecOps-project \
                        -Dsonar.organization=devsecops-project-key \
                        -Dsonar.host.url=https://sonarcloud.io \
                        -Dsonar.login=366a57a7a56bd77b0755f288c626bae1a1df59d8'
                }
            }
        }
        stage("Quality Gate") {
            steps {
              timeout(time: 15, unit: 'MINUTES') {
                waitForQualityGate abortPipeline: true
              }
            }
        }
        
        stage('Docker  Build') {
            steps {
      	        sh 'docker build -t ranmadrev7/sprint-boot-app:v1.$BUILD_ID .'
                sh 'docker image tag ranmadrev7/sprint-boot-app:v1.$BUILD_ID ranmadrev7/sprint-boot-app:latest'
            }
        }
        stage('Image Scan') {
            steps {
      	        sh ' trivy image --format template --template "@/usr/local/share/trivy/templates/html.tpl" -o report.html ranmadrev7/sprint-boot-app:latest '
            }
        }
        stage('Upload Scan report to AWS S3') {
              steps {
                  sh 'aws s3 cp report.html s3://devsecops-project-srm/'
              }
         }
        stage('Docker  Push') {
            steps {
                withVault(configuration: [skipSslVerification: true, timeout: 60, vaultCredentialId: 'vault-cred', vaultUrl: 'http://your-vault-server-ip:8200'], vaultSecrets: [[path: 'secrets/creds/docker', secretValues: [[vaultKey: 'username'], [vaultKey: 'password']]]]) {
                    sh "docker login -u ${username} -p ${password} "
                    sh 'docker push ranmadrev7/sprint-boot-app:v1.$BUILD_ID'
                    sh 'docker push ranmadrev7/sprint-boot-app:latest'
                    sh 'docker rmi ranmadrev7/sprint-boot-app:v1.$BUILD_ID ranmadrev7/sprint-boot-app:latest'
                }
            }
        }
        stage('Deploy to k8s') {
            steps {
                script{
                    kubernetesDeploy configs: 'spring-boot-deployment.yaml', kubeconfigId: 'kubernetes'
                }
            }
        }
        
 
    }
}
