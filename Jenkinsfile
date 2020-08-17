pipeline {
     agent any
     environment {
         AWS_REGION = 'us-east-2'
         AWS_DEFAULT_REGION = 'us-east-2'           
         }     
     stages {
         stage('Linting') {
              steps {
                sh 'tidy -q -e *.html'
              }
            }
            stage('Build image') {
              steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker_id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
                sh '''
                docker build -t sonubedi/udacitycapstone .
                '''
                }
              }
            }
            stage('Push Image To Dockerhub') {
              steps {
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'docker_id', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD']]){
                  sh '''
                    docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
                    docker push sonubedi/udacitycapstone:latest
                  '''
                }
              }
            }
            stage('Create conf file cluster') {
              steps {
                withAWS(region:'us-east-2', credentials:'AWS_ID') {
                  sh '''
                    aws eks --region us-east-2 update-kubeconfig --name capstone
                  '''
                }
              }
            }
            stage('Set current kubectl context') {
              steps {
                withAWS(region:'us-east-1', credentials:'AWS_ID') {
                  sh '''
                    kubectl config use-context arn:aws:eks:us-east-2:328932447510:cluster/capstone
                  '''
                }
              }
            }
            stage('Deploy blue container') {
              steps {
                withAWS(region:'us-east-2', credentials:'AWS_ID') {
                  sh '''
                    kubectl apply -f ./blue-controller.json
                  '''
                }
              }
            }

            stage('Deploy green container') {
              steps {
                withAWS(region:'us-east-2', credentials:'AWS_ID') {
                  sh '''
                    kubectl apply -f ./green-controller.json
                  '''
                }
              }
            }

            stage('Create the service in the cluster, redirect to blue') {
              steps {
                withAWS(region:'us-east-2', credentials:'AWS_ID') {
                  sh '''
                    kubectl apply -f ./blue-service.json
                  '''
                }
              }
            }

            stage('Wait user approve') {
                    steps {
                        input "Ready to redirect traffic to green?"
                    }
                }

            stage('Create the service in the cluster, redirect to green') {
              steps {
                withAWS(region:'us-east-2', credentials:'AWS_ID') {
                  sh '''
                    kubectl apply -f ./green-service.json
                  '''
                }
              }
            }

          }
        }