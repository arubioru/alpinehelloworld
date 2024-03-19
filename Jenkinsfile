pipeline {
     environment {
        PORT_EXPOSED = "80"
        DOCKERHUB_AUTH = credentials('DOCKERHUB_AUTH')
        ID_DOCKER = "${DOCKERHUB_AUTH_USR}"
     }

     agent none
     stages {

         stage('Build image') {
             agent any
             steps {
                script {
                  sh 'docker build -t ${votre_id_dockerhub}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
             }
        }
      
         stage('Run container based on builded image') {
            agent any
            steps {
               script {
                 sh '''
                    echo "Clean Env"
                    docker rm -f ${IMAGE_NAME} || echo "container does not exist"
                    docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${IMAGE_NAME}:${IMAGE_TAG}
                 '''
               }
            }
       }

       stage('Unit Test') {
           agent any
           steps {
              script {
                sh 'docker exec ${IMAGE_NAME} bash -c python test.py'
              }
           }
      }

      stage('Functioonal Test') {
           agent any
           steps {
              script {
                sh 'curl http://172.17.0.1 | grep -q "Hello World Antoine!"'
              }
           }
      }

      stage('Clean Container') {
          agent any
          steps {
             script {
               sh '''
                  docker stop ${IMAGE_NAME}
                  docker rm ${IMAGE_NAME}
              '''
             }
          }
     }

     stage ('Login and Push Image on docker hub') {
          agent any
        environment {
           DOCKERHUB_AUTH  = credentials('dockerhub')
        }            
          steps {
             script {
               sh '''
                  echo ${DOCKERHUB_PASSWORD_PSW} | docker login -u ${DOCKERHUB_AUTH_USER} --password-stdin
                  docker push ${ID_DOCKER}/${IMAGE_NAME}:${IMAGE_TAG}
               '''
             }
          }
      }

    stage('Deploy in staging') {
        agent any
        when {
            expression { GIT_BRANCH == 'origin/master' }
        }
        environment {
            DOCKERHUB_AUTH  = credentials('dockerhub')
            HOSTNAME_DEPLOY_STAGING = "35.153.54.246"
        }  
        steps {
          sshagent(credentials: ['SSH_AUTH_SERVER']) {
            sh '''
              [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
              ssh-keyscan -t rsa,dsa ${HOSTNAME_DEPLOY_STAGING} >> ~/.ssh/known_hosts
              command1="docker login -u ${DOCKER_AUTH_USER} -p ${DOCKER_AUTH_PSW}"
              command2="docker pull ${DOCKERHUB_AUTH_USR}/${IMAGE_NAME}:${IMAGE_TAG}"
              command3="docker rm ${IMAGE_NAME} || echo "app does not exist"
              command4="docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${DOCKERHUB_AUTH_USR}/${IMAGE_NAME}:${IMAGE_TAG}"
              ssh -t 
            '''
          }
        }
    }

    stage('Deploy in prod') {
        agent any
        when {
            expression { GIT_BRANCH == 'origin/master' }
        }
        environment {
            DOCKERHUB_AUTH  = credentials('dockerhub')
            HOSTNAME_DEPLOY_PROD = "54.205.162.45"
        }  
        steps {
          sshagent(credentials: ['SSH_AUTH_SERVER']) {
            sh '''
              [ -d ~/.ssh ] || mkdir ~/.ssh && chmod 0700 ~/.ssh
              ssh-keyscan -t rsa,dsa ${HOSTNAME_DEPLOY_STAGING} >> ~/.ssh/known_hosts
              command1="docker login -u ${DOCKER_AUTH_USER} -p ${DOCKER_AUTH_PSW}"
              command2="docker pull ${DOCKERHUB_AUTH_USR}/${IMAGE_NAME}:${IMAGE_TAG}"
              command3="docker rm ${IMAGE_NAME} || echo "app does not exist"
              command4="docker run -d -p 80:5000 -e PORT=5000 --name ${IMAGE_NAME} ${DOCKERHUB_AUTH_USR}/${IMAGE_NAME}:${IMAGE_TAG}"
              ssh -t centos@${HOSTNAME_DEPLOY_STAGING} \
                  -o SendEnv=IMAGE_NAME \
                  -o SendEnv=IMAGE_TAG \
                  -o SendEnv=DOCKERHUB_AUTH_USR \
                  -o SendEnv=DOCKERHUB_AUTH_PSW \
                  -C "${command1} && ${command2} && ${command3} && ${command4}"
            '''
          }
        }
    }

