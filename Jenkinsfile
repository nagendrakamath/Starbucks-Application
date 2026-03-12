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
        stage('Debug Docker Environment') {
            steps {
                // lightweight diagnostics to see where docker is available
                sh '''
                echo '--- which docker ---'
                which docker || echo 'docker not found on agent PATH'
                echo '--- docker --version ---'
                docker --version || true
                echo '--- ls /var/run/docker.sock ---'
                ls -l /var/run/docker.sock || true
                echo '--- uname & whoami ---'
                uname -a || true
                whoami || true
                echo '--- env DOCKER variables ---'
                env | grep -i docker || true
                '''
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
