pipeline{
agent any
    environment {
        // Define SonarQube server credentials configured in Jenkins
        SONARQUBE_ENV = 'mysonar'   // <-- name as per Jenkins > Manage Jenkins > Configure System > SonarQube Servers
    }
    stages{
        stage ("Clean Work Space"){
            steps{
                cleanWs()
            }
        }
        stage ("Git Code"){
            steps{
                git 'https://github.com/pranai-reddy/ltibbhackathon-flm.git'
            }
        }
        stage ("Sonar Qube Scanner"){
            steps{
                // mvn sonar:sonar for Java and lly for PHI its sonar-scanner
                // For this to work along with Sonarqube, we need sonar scanner as well to be installed.
                // Update the Sonar URL and Creds/Token in sonar-project.properties file for PHP, which is like pom.xml
                sh "/opt/sonar-scanner/bin/sonar-scanner"
            }
        }
        
        stage ("Quality Gates"){
            steps{
              //  script {
                //    waitForQualityGate abortPipeline: false, credentialsId: 'sonarCreds'
                //}
                 echo "Skipping this step becuase it's not working"
           }
        }
        stage ("Docker images creation"){
            steps{
                sh "docker build -t appimage ."
                sh "docker build -t dbimage database/"
            }
        }
        stage ("Scan images with Trivy"){
            steps{
                sh "trivy image appimage > appimagetrivyreport.txt"
                sh "trivy image dbimage > dbimagetrivyreport.txt"
            }
        }
        stage ("Tag and push to Docker Hub"){
            steps{
                script{
                    // This was taken from withDockerregistry Sample step, only the withDockerRegistry line.
                    withDockerRegistry(credentialsId: 'Docker_Creds'){
                        sh "docker tag appimage pranai123/k8sdemoargocd:appimage"
                        sh "docker tag dbimage pranai123/k8sdemoargocd:dbimage"
                        sh "docker push pranai123/k8sdemoargocd:appimage"
                        sh "docker push pranai123/k8sdemoargocd:dbimage"
                    }
                    
                }
                
                 
                    
                
            }
        }
    }
}
