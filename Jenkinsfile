pipeline{
    // agent any
    environment {
        IMAGE_NAME = 'alpinehelloworld'
        IMAGE_TAG = 'latest'
        // DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        // ID_DOCKER = 'malickfama'
        USER = 'malickfama'
        STAGING = 'env_staging'
        PRODUCTION = 'env_production'
        VPS_USER = 'ubuntu'                        // User de ton VPS
        VPS_IP = '1.2.3.4'                        // IP de ton VPS
    }
    agent none
    stages{

        stage('build image'){
            agent any
            steps{
                script{
                    sh "docker build -t ${USER}/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('run container'){
            agent any
            steps{
                script{
                    sh """
                         docker rm -f ${IMAGE_NAME} || true
                        docker run --name ${IMAGE_NAME} -d -p 8081:5000 -e PORT=5000  ${USER}/${IMAGE_NAME}:${IMAGE_TAG}
                        sleep 5s
                    """
                }
            }
        }

        stage('test'){
            agent any
            steps{
                script{
                    sh '''
                      sleep 5s 
                      curl http://host.docker.internal:8081 | grep -q "Hello world!" 
                    '''
                    // sh ' curl http://localhost:8081 | grep -q "Hello World!" '
                }
            }
        }

         stage('clean  containner'){
            agent any
            steps{
                script{
                    sh """
                        docker stop ${IMAGE_NAME} || true
                        docker rm ${IMAGE_NAME} || true

                   """
                }
            }
        }

        stage ('push image in Dockerhub'){
            agent any
            when {
                expression {env.GIT_BRANCH == 'origin/master'}
            }

            environment {
                DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
            }

            steps{
                script{
                    sh """
                    echo  $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                    docker push ${USER}/${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }

            }
        }

        stage ('deploy on Render'){
            agent=any
            when{
                expression{env.GIT_BRANCH== 'origin/master'}
            }
            environment{
                RENDER_SERVICE_ID =  'srv-d6j5sgfkijhs739c6l3g'
                RENDER_API_KEY =  credentials('render_api_key')
            }
            steps{
                script{
                    sh"""
                        curl -X POST https://api.render.com/v1/services/$RENDER_SERVICE_ID/deploys \
                        -H 'Authorization: Bearer $RENDER_API_KEY' \
                        -H 'Content-Type: application/json' \
                        -d '{"clearCache": false}'
                    """
                }
            }
        }

        stage ('deploy on VPS (production)'){
            when{
                expression{  env.GIT_BRANCH == 'origin/prod'}
            }

            environment{
                VPS_CREDENTIALS = credentials('vps-ssh-key')
            }

            steps{
                script{
                    sh """
                        ssh -o StrictHostKeyChecking=no -i $VPS_CREDENTIALS ${VPS_USER}@${VPS_IP } '
                            docker pull ${USER}/${IMAGE_NAME}:${IMAGE_TAG} &&
                            docker stop ${IMAGE_NAME} || true &&
                            docker rm ${IMAGE_NAME} || true &&
                            docker run -d --name ${IMAGE_NAME} -p 8081:5000 -e PORT=5000  ${USER}/${IMAGE_NAME}:${IMAGE_TAG}
                        ' 
                    """
                }
            }

        }

        stage ('push image in staging and deploy it'){
            when{
                    expression{env.GIT_BRANCH == 'origin/staging'}
                    // branch 'master'
            }
            agent any
            environment{
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps{
                script{
                    sh"""
                        heroku container:login
                        heroku create $STAGING  || echo "Env alreday exist"
                        heroku container:push -a $STAGING web
                        heroku container:release -a $STAGING web
                   """
                }
            }
        }

        stage ('push iamge in prod and deploy it'){

            when{
                expression{env.GIT_BRANCH == 'orgin/prod'}
                // branch 'master'
            }
            agent any
            environment{
                HEROKU_API_KEY = credentials('heroku_api_key')
            }

            steps{
                script{
                    sh """
                     heroku container:login
                     heroku create $PRODUCTION || echo " env exist"
                     heroku container:push -a $PRODUCTION web
                     heroku container:release -a $PRODUCTION web
                    """

                }
            } 
        }
    }
}
