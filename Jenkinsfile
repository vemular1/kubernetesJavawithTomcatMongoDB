pipeline {

    agent any

    environment {
       IMAGE = "gajjelli/kuberneteswithtomcatmongodb"
     }

    stages {

              stage ('cehckoutscm') {
                   steps {
                        checkout scm
                       }
                   }
               }

              stage ('mvn-compile.1') {
                      steps {
                          withMaven(maven : 'maven_3_5_2') {
                              sh 'mvn clean compile'
                          }
                      }
              }

              stage ('mvn-build.2') {
                      steps {
                          withMaven(maven : 'maven_3_5_2') {
                              sh 'mvn install'
                          }
                      }
              }

              stage('docker-build.3') {
                  agent any
                    steps {
                        script {
                          if ( env.BRANCH_NAME == 'master' ) {
                            pom = readMavenPom file: 'pom.xml'
                            TAG = pom.version
                            IMAGE = pom.name
                          } else {
                            TAG = env.BRANCH_NAME
                          }
                          sh "docker build -t ${IMAGE}:${TAG} ."
                        }
                    }
              }

              stage('docker-compose:start-local.4') {
                 agent any
                   steps {
                     sh 'docker-compose -f docker-compose.yml up'

                   }
              }

              stage('docker-compose:down-local.4') {
                   agent any
                     steps {
                       sh 'docker-compose -f docker-compose.yml down'
                     }
              }

             stage('push-dockerhub.6'){
                  steps {
                     withCredentials([usernamePassword(credentialsId: 'dockerHubAccount', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {

                           script {
                             if ( env.BRANCH_NAME == 'master' ) {
                                 pom = readMavenPom file: 'pom.xml'
                                 TAG = pom.version
                                 IMAGE = pom.name
                                } else {
                                TAG = env.BRANCH_NAME
                               }
                                 sh "docker login -u ${env.USERNAME} -p ${env.PASSWORD}"
                                 sh "docker tag ${IMAGE}:${TAG} ${env.USERNAME}/${IMAGE}:${TAG}"
                                 sh "docker push ${env.USERNAME}/${IMAGE}:${TAG}"
                                 echo "Image push complete."
                             }
                           }
                     }
            }
      }
}