pipeline {
  agent none
  
  environment {
    BUILD_TAG = "${env.TAG_PREFIX}-${env.BUILD_NUMBER.padLeft(5,'0')}"
  }

  stages {
    stage('Build') {
      agent { label 'docker-host' }
      steps {
        sendNotifications 'STARTED'
        withCredentials([usernamePassword(credentialsId: 'github-octavia-ci', passwordVariable: 'GIT_PASSWORD', usernameVariable: 'GIT_USERNAME')]) {
            sh "docker build -t securiport/kafka:${BUILD_TAG} ."
        }
        
      }
    }

    

    stage('Push to Nexus') {
        agent { label 'docker-host' }
        steps {
            withCredentials([usernamePassword(credentialsId: 'nexus', passwordVariable: 'NEXUS_PASSWORD', usernameVariable: 'NEXUS_USER')]) {
                script {
                    try {
                        sh  '''
                            docker login --username ${NEXUS_USER} --password ${NEXUS_PASSWORD} ${NEXUS_URL}
                            
                            docker tag securiport/kafka:${BUILD_TAG} ${NEXUS_URL}/securiport/kafka:${BUILD_TAG}
                            docker tag securiport/kafka:${BUILD_TAG} ${NEXUS_URL}/securiport/kafka:latest
                            
                            docker push ${NEXUS_URL}/securiport/kafka:${BUILD_TAG}
                            docker push ${NEXUS_URL}/securiport/kafka:latest
                           
                            '''
                    } catch(Exception e) {
                        currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
    }
  }
  
  post {
    always {
      sendNotifications currentBuild.result
    }
  }
}
