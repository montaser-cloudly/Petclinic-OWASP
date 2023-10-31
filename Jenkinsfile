pipeline {
    agent any
    environment {
    //    SLACK_CREDENTIAL = 'cloudly-slack-account'
    //    DOCKER_REGISTRY = "cloudlyio/petclinic"
        DOCKER_REGISTRY = "muntaseri/petclinic"
        SCANNER_HOME=tool 'omantel-nestJs'
    //    DOCKER_EC2_IP = '3.229.56.47'
    //    DOCKER_APP_NAME = 'health-status'
    }
    
    
    tools {
        jdk "jdk-18"
        maven "3.9.1"
        nodejs "node"
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
        
        stage('Build Artifact') {
            steps {
                sh "mvn clean package -DskipTests=true"
                //archive 'target/*.jar'
                archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
            }
        }
        
        stage('Test Maven JUnit') {
            steps {
                sh "mvn test"
            }
            post{
                always{
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }
        






        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('petclinic') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Petclinic-Montaser \
                    -Dsonar.java.binaries=. \
                    -Dsonar.projectKey=Petclinic-Montaser  \
                    -Dsonar.exclusions=target/site/jacoco/**/*.*
                       '''
                                                    
                }
            }
        }

/*
        stage("Quality Gate"){
            steps{
                script{
                    TEMP_STAGE_NAME=env.STAGE_NAME
                    timeout(time: 1, unit: 'HOURS') 
                    {
                        waitForQualityGate abortPipeline: true
                        def qg= waitForQualityGate()
                        if (qg.status!= 'OK'){
                            error "Pipeline aborted due to quality gate failure: ${qg.status}"
                        }
                    }         
                    echo 'Quality Gate Passed' 
                }
            }
        }
        
        
*/

//        stage("Quality Gate"){
//                steps{
//                timeout(time: 1, unit: 'HOURS') { // Just in case something goes wrong, pipeline will be killed after a timeout
//                def qg = waitForQualityGate() // Reuse taskId previously collected by withSonarQubeEnv
//                if (qg.status != 'OK') {
//                 error "Pipeline aborted due to quality gate failure: ${qg.status}"
//                }
//                }
//                }
//        }

        stage("OWASP Dependency Check"){
            steps{
                dependencyCheck additionalArguments: '--scan ./ --format XML ', odcInstallation: 'DP'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
                //dependencyCheckPublisher failedNewCritical: 15, failedTotalCritical: 20, pattern: 'dependency-check-report.xml', stopBuild: true, unstableNewCritical: 15, unstableTotalCritical: 20
                //dependencyCheckPublisher failedNewCritical: 50, failedTotalCritical: 60, pattern: 'dependency-check-report.xml', stopBuild: true, unstableNewCritical: 40, unstableTotalCritical: 50
                //dependencyCheckPublisher failedNewCritical: 10, failedNewHigh: 15, failedNewLow: 25, failedNewMedium: 20, failedTotalCritical: 15, failedTotalHigh: 20, failedTotalLow: 20, failedTotalMedium: 25, pattern: 'dependency-check-report.xml', stopBuild: true, unstableNewCritical: 5, unstableNewHigh: 10, unstableNewLow: 20, unstableNewMedium: 15, unstableTotalCritical: 10, unstableTotalHigh: 15, unstableTotalLow: 25, unstableTotalMedium: 20
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
                    withDockerRegistry(credentialsId: 'Dockerhub-montaser', toolName: 'docker')  {
                            sh "docker build -t petclinic ."
                            sh "docker tag petclinic $DOCKER_REGISTRY:v1"
                            sh "docker push $DOCKER_REGISTRY:v1"
                            }
                        }
                }
        }
   


    
        stage("TRIVY Image Scan"){
            steps{
            //script {
                    //withDockerRegistry(credentialsId: 'cloudly-dockerhub', toolName: 'docker')  {
                    //sh " trivy image $DOCKER_REGISTRY:v1"
                    
                    
                sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl'    
                sh 'mkdir -p reports'
                //sh 'trivy filesystem --ignore-unfixed --vuln-type os,library --format template --template "@html.tpl" -o reports/nodjs-scan.html ./nodejs'
                sh "trivy image  --format template --template '@html.tpl' -o reports/docker-image-scan.html $DOCKER_REGISTRY:v1"
                publishHTML target : [
                    allowMissing: true,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'docker-image-scan.html',
                    reportName: 'Trivy Image Scan',
                    reportTitles: 'Trivy Image Scan'
                ]

                // Scan again and fail on CRITICAL vulns
                //sh 'trivy filesystem --ignore-unfixed --vuln-type os,library --exit-code 1 --severity CRITICAL ./nodejs'
                
                // Scan again and fail on CRITICAL vulns
               // sh 'trivy image --exit-code 1 --severity CRITICAL  $DOCKER_REGISTRY:v1'

            }
        } 
        
                    
                    
                    
                    
                    
                    
            
        



// deploy

// deploy to docker

/*
stage("Deploy Using Docker"){
            steps{
                sh " docker run -d --name pet1 -p 8082:8082 writetoritika/pet-clinic123:latest "
            }
        }
        
  stage("Deploy To Tomcat"){
            steps{
                sh "cp  /var/lib/jenkins/workspace/Real-World-CI-CD/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/ "
            }
        }        
        

        // stage("Deploy To Tomcat"){
        //     steps{
        //         sh "cp  /var/lib/jenkins/workspace/CI-CD/target/petclinic.war /opt/apache-tomcat-9.0.65/webapps/ "
        //     }
        // }

*/



    
    }

}
