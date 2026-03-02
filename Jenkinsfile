pipline{
    agent any
    environment {
        IMAGE_NAME = 'alpinehelloworld'
        IMAGE_TAG = 'latest'
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials')
        ID_DOCKER = 'malickfama'
        USER = 'malickfama'
        STAGING = env_staging
        PRODUCTION = env_production
    }
     agent none
    stages{

        stage('build image'){
            agent any
            steps{
                script{
                    sh 'docker build -t ${USER}/${IMAGE_NAME}:${IMAGE_TAG} .'
                }
            }
        }

        stage('run container'){
            agent any
            steps{
                script{
                    sh '''
                        docker run --name ${IMAGE_NAME} -d -p 80:5000 -e PORT = 5000  ${USER}/${IMAGE_NAME}:${IMAGE:TAG}
                        sleep 5s
                    '''
                }
            }
        }

        stage('test'){
            agent any
            steps{
                script{
                    sh ' curl http://localhost:8080 | grep -q "Hello World!" '
                }
            }
        }

         stage('clean  containner'){
            agent any
            steps{
                script{
                    sh '''
                        docker stop ${IMAGE_NAME}
                        docker rm ${IMAGE_NAME}

                    '''
                }
            }
        }

        stage ('push image in staging and deploy it'){
            when{
                    expression{GIT_BRANCH == 'origin/master'}
            }
            agent any
            environment{
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps{
                script{
                    sh'''
                        heroku container:login
                        heroku create $STAGING  || echo "Env alreday exist"
                        heroku container:push -a $STAGING web
                        heroku container: release -a $STAGING web
                    '''
                }
            }
        }

        stage ('push iamge in prod and deploy it'){

            when{
                expression{'GIT_BRANCH == orgin/master'}
            }
            agent any
            environment{
                HEROKU_API_KEY = credentials('heroku_api_key')
            }

            steps{
                script{
                    sh '''
                     heroku container:login
                     heroku create $PRODUCTION || echo " env exist"
                     heroku container: push -a $PRODUCTION web
                     heroku container:release -a $PRODUCTION web
                    '''

                }
            } 
        }
    }
}