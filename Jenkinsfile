pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'tea-house'
        NEXUS_REGISTRY = 'nexus.imcc.com'
        NEXUS_REPO = 'repository/docker-hosted'
        SONAR_URL = 'http://sonarqube.imcc.com/'
        NEXUS_CREDENTIALS = credentials('nexus-credentials')
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('Build') {
            steps {
                container('dind') {
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Push to Nexus') {
            steps {
                container('dind') {
                    sh 'docker login $NEXUS_REGISTRY -u $NEXUS_CREDENTIALS_USR -p $NEXUS_CREDENTIALS_PSW'
                    sh 'docker tag $DOCKER_IMAGE $NEXUS_REGISTRY/$NEXUS_REPO/$DOCKER_IMAGE:latest'
                    sh 'docker push $NEXUS_REGISTRY/$NEXUS_REPO/$DOCKER_IMAGE:latest'
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                sh '''
                mkdir -p /tmp/bin
                curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
                chmod +x kubectl
                mv kubectl /tmp/bin/
                export PATH=$PATH:/tmp/bin
                kubectl apply -f k8s/
                '''
            }
        }
    }
}