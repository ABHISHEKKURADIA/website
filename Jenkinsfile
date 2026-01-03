pipeline {
    agent K8smaster

    environment {
        DOCKER_IMAGE = "abhishekkuradia/capstone-website"
        K8S_NODEPORT = "30008"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://abhishekkuradia.com/website.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${env.BUILD_ID} ."
                    sh "docker tag ${DOCKER_IMAGE}:${env.BUILD_ID} ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', passwordVariable: 'DOCKER_PASS', usernameVariable: 'DOCKER_USER')]) {
                    sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    sh "docker push ${DOCKER_IMAGE}:${env.BUILD_ID}"
                    sh "docker push ${DOCKER_IMAGE}:latest"
                }
            }
        }

        stage('Deploy to K8s (Date Check)') {
            steps {
                script {
                    // Check if today is the 25th
                    def today = new Date().format("dd")
                    if (today == "25") {
                        echo "Today is the 25th. Proceeding with Production Release..."
                        sh "kubectl apply -f k8s/deployment.yaml"
                        sh "kubectl apply -f k8s/service.yaml"
                    } else {
                        echo "Current day is ${today}. Release is restricted to the 25th. Skipping deployment."
                        // Optional: currentBuild.result = 'UNSTABLE'
                    }
                }
            }
        }
    }
}
