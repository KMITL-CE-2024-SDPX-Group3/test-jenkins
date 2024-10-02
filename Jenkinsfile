pipeline{
    agent any

    environment{
        APP_NAME = "SDPX - Test Jenkins"
        VENV_NAME = 'myenv'
        IMAGE_NAME = "ghcr.io/kmitl-ce-2024-sdpx-group3/test-jenkins-image"

    }

    stages {
        stage('Install Dependencies') {
            agent {
                label "Docker-Test-Node"
            }
            steps {
                // Install the dependencies for the Flask app
                sh 'python3 -m venv ${VENV_NAME}'
                sh '. ${VENV_NAME}/bin/activate && pip install -r requirements.txt'
            }
        }

        stage('Run Unit Tests') {
            agent {
                label "Docker-Test-Node"
            }
            steps {
                // Run the tests to validate the code
                sh '. ${VENV_NAME}/bin/activate && pytest --maxfail=1 --disable-warnings'
            }
        }

        stage('Build Docker Image') {
            agent {
                label "Docker-Test-Node"
            }
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                script {
                    // Build the Docker image for the Flask API
                    sh "docker compose up -d"
                }
            }
        }

        stage("Clone exam-api-robot repository"){
            agent {
                label "Docker-Test-Node"
            }
            steps {
                sh "git clone https://github.com/KMITL-CE-2024-SDPX-Group3/exam-api-robot"
            }
        }

        stage("Run Robot Test"){
            agent {
                label "Docker-Test-Node"
            }
            steps {
                sh ". ${VENV_NAME}/bin/activate && robot exam-api-robot/exam-app.robot"
            }
        }

        stage('Push to Docker Registry') {
            agent {
                label "Docker-Test-Node"
            }
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                script {
                    withCredentials(
                        [usernamePassword(
                            credentialsId: 'Sun-SDPX-Token', 
                            usernameVariable: 'GITHUB_USER', 
                            passwordVariable: 'GITHUB_PASSWORD'
                        )]
                    ){
                        sh "echo ${GITHUB_PASSWORD} | docker login ghcr.io -u ${GITHUB_USER} --password-stdin"
                    }
                    sh "docker compose push"
                }
            }
        }
        stage("Stop Docker Container and Remove Image"){
            agent {
                label "Docker-Test-Node"
            }
            steps {
                sh "docker compose down --rmi local"
            }
        }
    }

    post {
        always {
            // Clean up Docker images and other resources
            sh "docker rmi ${DOCKER_IMAGE}:latest || true"
            sh "rm -rf ${VENV_NAME}"
            sh "rm -rf exam-api-robot"
        }
    }
}