pipeline {
    agent any
    environment { 
      DOCKER_ID = "matthieuportelli"
      DOCKER_IMAGE = "datascientestapi"
      DOCKER_TAG = "v.${BUILD_ID}.0" 
    }
    stages {
        stage('Building') {
            steps {
                  sh 'pip install -r requirements.txt'
            }
        }
        stage('Testing') {
            steps {
                sh 'python3 -m unittest'
            }
        }
        stage('Deploying') {
            steps{
                script {
                    sh '''
                    docker rm -f jenkins
                    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
                    docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
                    '''
                }
            }
        }
        stage ('User acceptance'){
            steps{
                input(message: "Proceed to push to main", ok: "Yes")
            }
        }
        stage('Pushing and Merging') {
            parallel {
                stage('Pushing') {
                    environment {
                        DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_PASS')
                    }
                    steps {
                        sh 'echo $DOCKERHUB_CREDENTIALS | docker login -u $DOCKER_ID --password-stdin'
                        sh 'docker image push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG'
                    }
                }
                stage('Merging') {
                    steps {
                        echo 'Merging done'
                    }
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}