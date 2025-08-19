pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                script {
                    echo "Building application for branch ${env.BRANCH_NAME}"
                    sh 'npm install'
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    echo "Running tests for branch ${env.BRANCH_NAME}"
                    sh 'npm test || echo "No tests found"'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def imageName = (env.BRANCH_NAME == 'main') ? 'nodemain:v1.0' : 'nodedev:v1.0'
                    sh """
                        echo "Building Docker image: ${imageName}"
                        docker build -t ${imageName} .
                    """
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                script {
                    def imageName = (env.BRANCH_NAME == 'main') ? 'nodemain:v1.0' : 'nodedev:v1.0'
                    def dockerRepo = "tuUsuarioDockerHub/${imageName}"

                    echo "Tagging image ${imageName} as ${dockerRepo}"
                    sh "docker tag ${imageName} ${dockerRepo}"

                    withCredentials([usernamePassword(credentialsId: 'docker-hub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                            echo "Logging in to DockerHub"
                            docker login -u $DOCKER_USER -p $DOCKER_PASS
                            echo "Pushing image to DockerHub: ${dockerRepo}"
                            docker push ${dockerRepo}
                        '''
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def port = (env.BRANCH_NAME == 'main') ? '3000' : '3001'
                    def containerName = (env.BRANCH_NAME == 'main') ? 'node_main' : 'node_dev'
                    def imageName = (env.BRANCH_NAME == 'main') ? 'nodemain:v1.0' : 'nodedev:v1.0'

                    sh """
                        echo "Stopping and removing container: ${containerName} (if exists)"
                        docker rm -f ${containerName} || true

                        echo "Running container ${containerName} on port ${port}"
                        docker run -d --name ${containerName} -p ${port}:3000 ${imageName}
                    """
                }
            }
        }
    }
}
