@Library('my-shared-library') _

pipeline{

    agent{
      docker{
        image 'yash6370/mvn-trivy-agent'
        args '--user root -v /var/run/docker.sock:/var/run/docker.sock' // mount Docker socket to access the host's Docker daemon
      }
    } 

    parameters{
        choice(name: 'action', choices: 'create\ndelete', description: 'Choose Create/Delete')
        string(name: 'ImageName', description: "Name of the docker image", defaultValue: 'simple_java_app')
        string(name: 'ImageTag', description: "Tag of the docker image", defaultValue: 'v1')
        string(name: 'DockerHubUser', description: "Docker Hub username", defaultValue: 'yash91989201')
    }

    stages{
        
        stage("check versions"){
          steps{
                      sh 'java --version'
          sh 'mvn -v'

          }
        }
        stage('Git Checkout'){
            when { expression {  params.action == 'create' } }
            steps {
            gitCheckout(
                branch: "main",
                url: "https://github.com/yash91989201/mrdevops_java_app.git"
            )
            }
        }
         stage('Unit Test maven'){
         
         when { expression {  params.action == 'create' } }

            steps{
               script{
                   
                   mvnTest()
               }
            }
        }
         stage('Integration Test maven'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   mvnIntegrationTest()
               }
            }
        }
        stage('Static code analysis: Sonarqube'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   def SonarQubecredentialsId = 'sonarqube-creds'
                   statiCodeAnalysis(SonarQubecredentialsId)
               }
            }
        }
        stage('Quality Gate Status Check : Sonarqube'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   def SonarQubecredentialsId = 'sonarqube-creds'
                   QualityGateStatus(SonarQubecredentialsId)
               }
            }
        }
        stage('Maven Build : maven'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   mvnBuild()
               }
            }
        }
        stage('Docker Image Build'){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   dockerBuild("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }
         stage('Docker Image Scan: trivy '){
         when { expression {  params.action == 'create' } }
            steps{
              sh '''

              apt-get update \
  && apt-get install -y apt-transport-https gnupg lsb-release \
  && wget -qO - https://aquasecurity.github.io/trivy-repo/deb/public.key | gpg --dearmor | tee /usr/share/keyrings/trivy.gpg > /dev/null \
  && echo "deb [signed-by=/usr/share/keyrings/trivy.gpg] https://aquasecurity.github.io/trivy-repo/deb $(lsb_release -sc) main" | tee -a /etc/apt/sources.list.d/trivy.list \
  && apt-get update \
  && apt-get install -y trivy
              '''
               script{
                   
                   dockerImageScan("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }
        stage('Docker Image Push : DockerHub '){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   dockerImagePush("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }   
        stage('Docker Image Cleanup : DockerHub '){
         when { expression {  params.action == 'create' } }
            steps{
               script{
                   
                   dockerImageCleanup("${params.ImageName}","${params.ImageTag}","${params.DockerHubUser}")
               }
            }
        }      
    }
}
