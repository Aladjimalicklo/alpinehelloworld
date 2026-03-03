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

        stage ('push image in staging and deploy it'){
            when{
                    expression{env.GIT_BRANCH == 'origin/master'}
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
                        heroku container: release -a $STAGING web
                   """
                }
            }
        }

        stage ('push iamge in prod and deploy it'){

            when{
                expression{env.GIT_BRANCH == 'orgin/master'}
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
                     heroku container: push -a $PRODUCTION web
                     heroku container:release -a $PRODUCTION web
                    """

                }
            } 
        }
    }
}
