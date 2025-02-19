pipeline {
    environment {
        IMAGE_REPO = "stephanecursan"
        IMAGE_NAME = "alpinehelloworld"
        IMAGE_TAG = "latest"
        STAGING = "stephanecursan-staging"
        PRODUCTION = "stephanecursan-production"
    }
    agent none
    stages {
        stage('build image') {
            agent any
            steps {
                script {
                    sh 'docker build -t $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }
        stage('run container based on builded image') {
            agent any
            steps {
                script {
                    sh '''
                        docker run --name $IMAGE_NAME -p 80:5000 -e PORT=5000 -d $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG
                        sleep 5
                    '''
                }
            }
        }
        stage('test image') {
            agent any
            steps {
                script {
                    sh 'curl http://172.17.0.1 | grep -q "Hello world"'
                }
            }
        }
        stage('clean container') {
            agent any
            steps {
                script {
                    sh 'docker rm -vf $IMAGE_NAME'
                }
            }
        }
        stage('push image on dockerhub') {
            agent any
            environment {
                DOCKERHUB_LOGIN = credentials('dockerhub_stephanecursan')
            }
            steps {
                script {
                     sh '''
                         docker login --username $DOCKERHUB_LOGIN_USR --password $DOCKERHUB_LOGIN_PSW
                         docker push $IMAGE_REPO/$IMAGE_NAME:$IMAGE_TAG
                     '''
                }
            }
        }
        stage('push image in staging and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                        heroku container:login
                        heroku create $STAGING || echo "project already exist"
                        heroku container:push -a $STAGING web
                        heroku container:release -a $STAGING web
                    '''
                }
            }
        }
        stage('test staging deployment') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            agent any
            steps {
                script {
                     sh 'curl https://$STAGING.herokuapp.com | grep -q "Hello world"'
                }
            }
        }
        stage('push image in production and deploy it') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            agent any
            environment {
                HEROKU_API_KEY = credentials('heroku_api_key')
            }
            steps {
                script {
                    sh '''
                        heroku container:login
                        heroku create $PRODUCTION || echo "project already exist"
                        heroku container:push -a $PRODUCTION web
                        heroku container:release -a $PRODUCTION web
                    '''
                }
            }
        }
        stage('test production deployment') {
            when {
                expression { GIT_BRANCH == 'origin/master' }
            }
            agent any
            steps {
                script {
                     sh 'curl https://$PRODUCTION.herokuapp.com | grep -q "Hello world"'
                }
            }
        }
    }
}
