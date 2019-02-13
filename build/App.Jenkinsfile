#!groovy
     
    pipeline {
      agent {
        label 'host'
      }
      
      options {
        buildDiscarder(logRotator(numToKeepStr: '10', daysToKeepStr: '7'))
        disableConcurrentBuilds()
      }
      
      stages {
     
        stage("Set Build Parameters") {
          steps {
            script {
              currentBuild.displayName = "Build_App .${BUILD_NUMBER}";
            }
          }
        }
     
        stage("Build") {
          steps {
            sh 'docker run --rm --name WebGoat -v "$(pwd)":/usr/src/mymaven -w /usr/src/mymaven maven mvn clean install'
          }
        }
     
    	stage("Dependency Check") {
          steps {
            sh 'docker run -v ${PWD}:/src -v ${PWD}/result/:/result melaniealwardt/dependency-check:latest --scan /src/ --format "ALL" --project WebGoat --out /result'
          }
        }
     
        stage("Code Quality") {
          steps {
            dir("./") {
              withSonarQubeEnv('Sonarqube') {
                withCredentials([usernamePassword(credentialsId: 'lidop', passwordVariable: 'rootPassword', usernameVariable: 'rootUser')]) {
                  sh 'docker run --dns ${IPADDRESS} --rm  -v ${PWD}/:/work -e SERVER=http://sonarqube.service.lidop.local:8084/sonarqube -e PROJECT_KEY=helloworldnodejs  registry.service.lidop.local:5000/lidop/sonarscanner:latest'
                }
              }
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
              }
     
            }
          }
        }
              
      }
     
      post { 
        always { 
          script {
            currentBuild.description = "goto <a href=https://www.${PUBLIC_IPADDRESS}.xip.io/port/9100/>App</a>"
            try {
              sh "docker rm -f helloworldnodejs-unittest"
            }
            catch(err) {
              echo "No running Container"
            }
          }
        }
      }
     
    }