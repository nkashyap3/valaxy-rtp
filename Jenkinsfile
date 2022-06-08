def imageName = 'nkashyap.jfrog.io/artifactory/default-docker-local/valaxy-rtp'
def registry  = 'https://nkashyap.jfrog.io/artifactory'
def version   = '1.0.3'
def app
pipeline {
    agent {
       node {
         label "valaxy"
      }
    }
    stages {
        stage('Build') {
            steps {
                echo '<--------------- Building --------------->'
                sh 'printenv'
                sh 'mvn clean deploy -Dmaven.test.skip=true'
                echo '<------------- Build completed --------------->'
            }
        }

        stage('Unit Test') {
            steps {
                echo '<--------------- Unit Test Start --------------->'
                sh 'mvn surefire-report:report'
                echo '<------------- Unit Test completed --------------->'
            }
        }        

        stage('Sonar Analysis'){
            environment{
                scannerHome = tool 'SonarQubeScanner'
            }
           steps{
               echo '-----Sonar-Analysis-Started-----'
               withSonarQubeEnv('SonarQube'){
                sh "${scannerHome}/bin/sonar-scanner"
               }
               echo '-----Sonar-Analysis-Ends-----'
           }
        } 

        stage("Quality Gate") {
            steps {
                script {
                  echo '<--------------- Sonar Gate Analysis Started --------------->'
                    timeout(time: 1, unit: 'HOURS'){
                       def qg = waitForQualityGate()
                        if(qg.status !='OK') {
                            error "Pipeline failed due to quality gate failures: ${qg.status}"
                        }
                    }  
                  echo '<--------------- Sonar Gate Analysis Ends  --------------->'
                }
            }
        }//Quality Gate Stage ENDS//

        stage('Docker Build'){
           steps{
             script{
               echo '-----Docker-Build-Started-----'
               app = docker.build(imageName)
               echo '-----Docker-Build-Ends-----'
           }
        }         
      }//Docker Build Stage ENDS

        stage('Jar Publish'){
           steps{
             script{
               echo '-----JAR-Publish-Started-----'
               def server = Artifactory.newServer url:registry ,  credentialsId:"artifactcredid"
                def properties = "buildid=${env.BUILD_ID},commitid=${GIT_COMMIT}";
                def uploadSpec = """{
                      "files": [
                        {
                          "pattern": "jarstaging/(*)",
                          "target": "default-maven-local/{1}",
                          "flat": "false",
                          "props" : "${properties}",
                          "exclusions": [ "*.sha1", "*.md5"]
                        }
                     ]
                 }"""
                def buildInfo = server.upload(uploadSpec)
                buildInfo.env.collect()
                server.publishBuildInfo(buildInfo)
               echo '-----JAR-Publish-Ends-----'
           }
        }         
      }// JAR PUBLISH STAGE ENDS

        stage("Docker Publish") {
          steps {
            script {
               echo '<--------------- Docker Publish Started --------------->'
               docker.withRegistry(registry , 'artifactcredid'){
                 docker.image(imageName).push(version)
               }
               echo '<--------------- Docker Publish Ends --------------->'
            }
          }
        }  //Docker Publish stage ends

    }
 }