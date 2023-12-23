pipeline{
    agent any 
    tools{                                              
        jdk 'jdk17'
        nodejs 'node16'
    }
    environment{

        scannerHome= tool 'sonar-scanner'
        //API= credentials('api')  
    }

    stages{

        stage("clean workspace"){
            steps{
                cleanWs()
            }    
        }


        stage('Checkout from Git'){
            steps{
                git branch: 'main', url: 'https://github.com/mustafa69-code/devsec.git'
            }
        }

        stage("sonarqube analysis"){
            steps{
                withSonarQubeEnv('sonar-server') {    
                sh '''
                 ${scannerHome}/bin/sonar-scanner \
                 -Dsonar.projectName=myproject -Dsonar.projectKey=myproject
                 '''
              }
            }    
        }

        stage("Quality Gate ") {
            steps {
                script{
                     waitForQualityGate abortPipeline: false, credentialsId: 'Sonar-token' 
                }
               
            }
        }

        stage("install dependency"){
            steps{
                sh 'npm install'
            }     
        }
        
        stage("OWASP fs scan"){
            steps{
              dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
              dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
              
            }
        }

        stage("Trivy fs scan"){
            steps{
              sh 'trivy fs . > trivyoutput.txt'     
            }
        }

        stage("build the image"){
            steps{
              script{
                withDockerRegistry(credentialsId: 'docker', toolName: 'docker') {
                
                   sh "docker build --build-arg TMDB_V3_API_KEY=180196a1672216ae6d62be15840e4cde -t netflix ." 
                   sh "docker tag  netflix mustafa6969/netflix:latest"
                   sh "docker push mustafa6969/netflix:latest"
                }
              }
            } 
        }


        stage("trivy image"){
            steps{

                sh 'trivy image mustafa6969/netflix:latest > trivyimage.txt'
                
            }
        }   

        stage("deploy the app"){
            steps{
              sh 'docker run --name netflix -dp 8081:80 mustafa6969/netflix:latest' 
              
            }
        }

    }


}
   
    


















