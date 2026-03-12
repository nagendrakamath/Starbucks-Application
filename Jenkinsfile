pipeline {
    agent any
    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }
    stages {
        stage ("Clean Workspace") {
            steps {
                cleanWs()
            }
        }
        stage ("Git Checkout") {
            steps {
                git branch: 'main', url: 'https://github.com/nagendrakamath/Starbucks-Application.git'
            }
        }
        stage("Install NPM Dependencies") {
            steps {
                sh "npm install"
            }
        }
        stage("Build Docker Image") {
            // run this stage inside a container that has the docker CLI and access to host docker daemon
            agent {
                docker {
                    image 'docker:24.0.2'
                    // allow the container to talk to the host docker daemon
                    args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                sh 'docker build -t starbucks .'
            }
        }
        stage("Tag & Push to DockerHub") {
            agent {
                docker {
                    image 'docker:24.0.2'
                    args '--privileged -v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker') {
                        sh 'docker tag starbucks nagendrakamath/starbucks:latest'
                        sh 'docker push nagendrakamath/starbucks:latest'
                    }
                }
            }
        }
    }
}
