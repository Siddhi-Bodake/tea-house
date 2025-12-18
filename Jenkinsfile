pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: sonar-scanner
    image: sonarsource/sonar-scanner-cli
    command: ["cat"]
    tty: true

  - name: kubectl
    image: bitnami/kubectl:latest
    command: ["cat"]
    tty: true
    securityContext:
      runAsUser: 0
      readOnlyRootFilesystem: false
    env:
    - name: KUBECONFIG
      value: /kube/config
    volumeMounts:
    - name: kubeconfig-secret
      mountPath: /kube/config
      subPath: kubeconfig

  - name: dind
    image: docker:dind
    securityContext:
      privileged: true
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ""

  volumes:
  - name: kubeconfig-secret
    secret:
      secretName: kubeconfig-secret
'''
        }
    }

    environment {
        DOCKER_IMAGE = 'tea-house'
        NEXUS_REGISTRY = 'nexus.imcc.com'
        NEXUS_REPO = 'repository/docker-hosted'
        SONAR_HOST = 'http://sonarqube.imcc.com/'
        NEXUS_CREDENTIALS = credentials('nexus-credentials')
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {
        stage('Build Docker Image') {
            steps {
                container('dind') {
                    sh '''
                        echo "Building Tea House Docker image..."
                        sleep 15
                        docker build -t $DOCKER_IMAGE:latest .
                    '''
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                container('sonar-scanner') {
                    sh '''
                        sonar-scanner \
                          -Dsonar.projectKey=tea-house \
                          -Dsonar.sources=. \
                          -Dsonar.exclusions=node_modules/**,dist/** \
                          -Dsonar.host.url=${SONAR_HOST} \
                          -Dsonar.token=${SONAR_TOKEN}
                    '''
                }
            }
        }

        stage('Login to Nexus Registry') {
            steps {
                container('dind') {
                    sh '''
                        docker login $NEXUS_REGISTRY \
                          -u $NEXUS_CREDENTIALS_USR -p $NEXUS_CREDENTIALS_PSW
                    '''
                }
            }
        }

        stage('Push Tea House Image to Nexus') {
            steps {
                container('dind') {
                    sh '''
                        docker tag $DOCKER_IMAGE:latest \
                          $NEXUS_REGISTRY/$NEXUS_REPO/$DOCKER_IMAGE:v1

                        docker push \
                          $NEXUS_REGISTRY/$NEXUS_REPO/$DOCKER_IMAGE:v1

                        docker image ls
                    '''
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    sh '''
                        echo "Applying Tea House Kubernetes Deployment & Service..."

                        kubectl apply -f k8s/deployment.yaml
                        kubectl apply -f k8s/service.yaml

                        echo "Checking all resources..."
                        kubectl get all

                        echo "Waiting for rollout..."
                        kubectl rollout status deployment/tea-house --timeout=120s
                    '''
                }
            }
        }
    }
}