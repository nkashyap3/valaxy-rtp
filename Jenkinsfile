def imageName = 'stalin.jfrog.io/default-docker-local/valaxy-rtp'
def registry  = 'https://stalin.jfrog.io'
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

        stage('Quality Gate Analysis'){
              script{
                echo '-----Quality-Gate-Analysis-Starts----'
                timeout(time:1, unit: 'HOURS'){
                def qg = waitForQualityGate()
                if ( qg.status != 'OK'){
                    error "Pipeline failed due to Quality Gates failure: ${qg.status}"
                              }
                }
                echo '-----Quality-Gate-Analysis-Ends-----'
              }
           }
        }
    }
 }
