pipeline {
    agent {
        kubernetes {
            yaml '''
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins: docker-build-agent
spec:
  containers:
  - name: docker
    image: docker:27-cli
    command:
    - cat
    tty: true
    env:
    - name: DOCKER_HOST
      value: tcp://localhost:2375
  - name: dind
    image: docker:27-dind
    securityContext:
      privileged: true
    env:
    - name: DOCKER_TLS_CERTDIR
      value: ""
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
'''
        }
    }

    environment {
        DOCKERHUB_USER = 'sghazala'
        APP_IMAGE = 'vprofileapp'
        DB_IMAGE = 'vprofiledb'
        TAG = 'latest'
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/samahghazala/iti_cicd.git'
            }
        }

        stage('Build App Image') {
            steps {
                container('docker') {
                    script {
                        dir('Docker-files/app') {
                            sh "docker build --build-arg BUILD_NUMBER=${env.BUILD_NUMBER} -t ${DOCKERHUB_USER}/${APP_IMAGE}:${TAG} ."
                        }
                    }
                }
            }
        }

        stage('Build DB Image') {
            steps {
                container('docker') {
                    script {
                        dir('Docker-files/db') {
                            sh "docker build -t ${DOCKERHUB_USER}/${DB_IMAGE}:${TAG} ."
                        }
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                container('docker') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh '''
                            echo "$PASS" | docker login -u "$USER" --password-stdin
                            docker push ${DOCKERHUB_USER}/${APP_IMAGE}:${TAG}
                            docker push ${DOCKERHUB_USER}/${DB_IMAGE}:${TAG}
                            docker logout
                        '''
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                container('kubectl') {
                    withKubeConfig(serverUrl: 'https://cluster.local') {
                        sh '''
                            kubectl apply -f app-secret.yml
                            kubectl apply -f db-CIP.yml
                            kubectl apply -f mc-CIP.yml
                            kubectl apply -f mcdep.yml
                            kubectl apply -f rmq-CIP-service.yml
                            kubectl apply -f rmq-dep.yml
                            kubectl apply -f vproapp-service.yml
                            kubectl apply -f vproappdep.yml
                            kubectl apply -f vprodbdep.yml
                            
                            kubectl rollout restart deployment/vproappdep || true
                            kubectl rollout restart deployment/vprodbdep || true
                        '''
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build and deployment completed successfully."
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
