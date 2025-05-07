pipeline {
    agent any

    environment {
        DOCKER_IMAGE_NAME = 'apurwasingh/flask'
        DOCKER_IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

    stage('Test') {
    steps {
        script {
            docker.build("test-image")
            sh 'docker run -d --rm --name test_flask --network jenkins_net test-image'
            sleep 5

            // Check if container is still running
            def running = sh(script: 'docker ps -q -f name=test_flask', returnStdout: true).trim()
            if (!running) {
                error "test_flask container exited early. Likely app.py has errors."
            }

            // Now do the curl check
            sh 'docker run --rm --network jenkins_net curlimages/curl:latest curl -f http://test_flask:7000'

            // Clean up
            sh 'docker stop test_flask || true'
        }
    }
}

stage('Cleanup') {
    steps {
        script {
            // Remove dangling images
            sh 'docker image prune -f'

 
        }
    }
}

        
        stage('Build & Push') {
            steps {
                script {
                    docker.build("${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}")
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-hub-credentials') {
                        docker.image("${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}").push()
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

