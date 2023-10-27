pipeline {
    agent any
    environment {
    //    SLACK_CREDENTIAL = 'cloudly-slack-account'
        DOCKER_REGISTRY = "cloudlyio/petclinic"
        SCANNER_HOME=tool 'omantel-nestJs'
    //    DOCKER_EC2_IP = '3.229.56.47'
    //    DOCKER_APP_NAME = 'health-status'
    }
    
    
    tools {
        jdk "jdk-18"
        maven "3.9.1"
    }
    
    
    
    stages {
        stage('git-pull') {
            steps {
                git branch: 'main', credentialsId: 'github-montaser-cloudly', url: 'https://github.com/montaser-cloudly/Petclinic-OWASP.git'
            }
        }
        
        stage('Code Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        
        stage('unit test') {
            steps {
                sh "mvn test"
            }
        }
        
        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('petclinic') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=PetclinicMasum \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=PetclinicMasum '''
    
                }
            }
        }

        stage("Quality Gate"){
        timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
         def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
            if (qg.status != 'OK') {
             error "Pipeline aborted due to quality gate failure: ${qg.status}"
                }
            }
        }


        
        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format xml ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
        
        stage("Build"){
            steps{
                sh " mvn clean install"
            }
        }
  

        stage("Docker Build"){
            steps{
                script {
                    withDockerRegistry(credentialsId: 'cloudly-dockerhub', toolName: 'docker')  {
                            sh "docker build -t petclinic ."
                            sh "docker tag petclinic cloudlyio/petclinic:v1"
                            sh "docker push cloudlyio/petclinic:v1"
                            }
                        }
                }
        }
   


    
        stage("TRIVY Image Scan"){
            steps{
                sh " trivy image $DOCKER_REGISTRY:v1"
            }
        }



// deploy



        // stage("Deploy To Tomcat"){
        //     steps{
        //         sh "cp  /var/lib/jenkins/workspace/CI-CD/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/ "
        //     }
        // }


    
    }

}
