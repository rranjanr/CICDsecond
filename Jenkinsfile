pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'rranzan2001/flask'
        DOCKER_IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Create Network') {
            steps {
                script {
                    sh '''
                        docker network ls | grep jenkins_net || docker network create jenkins_net
                    '''
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    // 🔁 Ensure no leftover container before build
                    sh 'docker rm -f test_flask || true'

                    docker.build("test-image")

                    try {
                        // 🐳 Run container for testing
                        sh 'docker run -d --name test_flask --network jenkins_net test-image'
                        sleep 5

                        // ✅ Verify container is running
                        def running = sh(script: 'docker ps -q -f name=test_flask', returnStdout: true).trim()
                        if (!running) {
                            error "test_flask container exited early. Likely app.py has errors."
                        }

                        // 🌐 Health check via curl
                        sh 'docker run --rm --network jenkins_net curlimages/curl:latest curl -f http://test_flask:7000'
                    } finally {
                        // 🧹 Clean up after test
                        sh 'docker stop test_flask || true'
                    }
                }
            }
            post {
                always {
                    // 🔁 Ensure container is removed in all outcomes
                    sh 'docker rm -f test_flask || true'
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    sh 'docker image prune -f'
                }
            }
        }

        stage('Build & Push') {
            steps {
                script {
                    def image = docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}")
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        image.push()
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh "docker exec -u root ansible ansible-playbook /root/deploy.yml"
                }
            }
        }
    }
}
