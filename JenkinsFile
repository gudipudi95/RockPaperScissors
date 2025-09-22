pipeline{
    agent { label 'Built-In Node'}
    environment {
        REGISTRY="pragu095"
        IMAGE_NAME="project1"
    }
    stages{
        stage('Checkout'){
            steps{
                checkout scm
            }
        }

        stage('Build Docker Image'){
            steps{
                script {
                    //sh "docker build -t $REGISTRY/$IMAGE_NAME:${env.BUILD_NUMBER} ."
                    IMAGE_TAG = "$REGISTRY/$IMAGE_NAME:${env.BRANCH_NAME}-${env.BUILD_NUMBER}"
                    IMAGE_LATEST = "$REGISTRY/$IMAGE_NAME:latest-${env.BRANCH_NAME}"
                        sh "docker build -t $IMAGE_TAG ."
                        sh "docker tag $IMAGE_TAG $IMAGE_LATEST"
                }
            }
        }
        stage('push docker image'){
            steps{
                withCredentials([usernamePassword(credentialsId:'dockerhub-creds',
                usernameVariable: 'DOCKER_USER',passwordVariable: 'DOCKER_PASS')]){
                sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                sh "docker push  $IMAGE_TAG"
                sh "docker push  $IMAGE_LATEST"
                stash name: 'src', includes: '**/*'
                }
            }
        }

        stage('Deploy'){
            steps{
                script{
                    if(env.BRANCH_NAME=='dev'){
                        node('dev-test-node'){
                            unstash 'src'
                            withCredentials([usernamePassword(credentialsId:'dockerhub-creds',
                usernameVariable: 'DOCKER_USER',passwordVariable: 'DOCKER_PASS')]){
                sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                            sh"""
                            docker service rm dev_stack || true
                            docker pull pragu095/project1:latest-dev
                            docker stack deploy -c docker-compose.dev.yaml dev_stack
                            """
                            }
                        }
                    }else if(env.BRANCH_NAME=='test'){
                        node('dev-test-node'){
                            unstash 'src'
                            withCredentials([usernamePassword(credentialsId:'dockerhub-creds',
                usernameVariable: 'DOCKER_USER',passwordVariable: 'DOCKER_PASS')]){
                sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                            sh"""
                            docker service rm test_stack || true
                            docker pull pragu095/project1:latest-test
                            docker stack deploy -c docker-compose.test.yaml test_stack
                            """
                            }
                        }
                    }else if(env.BRANCH_NAME=='main'){
                        node('prod-node'){
                            withCredentials([usernamePassword(credentialsId:'dockerhub-creds',
                usernameVariable: 'DOCKER_USER',passwordVariable: 'DOCKER_PASS')]){
                sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                            unstash 'src'
                            sh"""
                            docker service rm prod_stack || true
                            docker pull pragu095/project1:latest-prod
                            docker stack deploy -c docker-compose.prod.yaml prod_stack
                            """
                            }
                        }
                    }
                }
            }
        }
    }
}
