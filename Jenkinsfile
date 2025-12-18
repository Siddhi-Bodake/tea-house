pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'tea-house'
        NEXUS_URL = 'http://nexus.imcc.com/'
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

        stage('SonarQube Scan') {
            steps {
                sh 'curl -sSLo /tmp/sonar-scanner.zip https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.8.0.2856-linux.zip'
                sh 'unzip -o /tmp/sonar-scanner.zip -d /tmp/'
                sh 'export PATH=$PATH:/tmp/sonar-scanner-4.8.0.2856-linux/bin && sonar-scanner'
            }
        }

        stage('Push to Nexus') {
            steps {
                container('dind') {
                    sh 'echo $NEXUS_CREDENTIALS_PSW | docker login $NEXUS_URL -u $NEXUS_CREDENTIALS_USR --password-stdin'
                    sh 'docker tag $DOCKER_IMAGE $NEXUS_URL/repository/docker-hosted/$DOCKER_IMAGE:latest'
                    sh 'docker push $NEXUS_URL/repository/docker-hosted/$DOCKER_IMAGE:latest'
                }
            }
        }

        stage('Deploy to K8s') {
            steps {
                sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                sh 'chmod +x kubectl && mv kubectl /usr/local/bin/'
                sh 'kubectl apply -f k8s/'
            }
        }
    }
}