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
  nodeName: controlplane
  containers:
  - name: docker
    image: docker:27-cli
    command:
    - cat
    tty: true
    securityContext:
      runAsUser: 0
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  - name: kubectl
    image: bitnami/kubectl:latest
    command:
    - cat
    tty: true
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
'''
        }
    }
 environment {
    DOCKERHUB_USER = 'sghazala'
    APP_IMAGE      = 'vprofileapp'
    DB_IMAGE       = 'vprofiledb'
    TAG            = "${env.BUILD_NUMBER}"
}

stages {
    stage('Build App Image') {
        steps {
            container('docker') {
                script {
                    dir('Docker-files/app') {
                        sh 'docker build --build-arg BUILD_NUMBER=$TAG -t $DOCKERHUB_USER/$APP_IMAGE:$TAG -t $DOCKERHUB_USER/$APP_IMAGE:latest .'
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
                        sh 'docker build -t $DOCKERHUB_USER/$DB_IMAGE:$TAG -t $DOCKERHUB_USER/$DB_IMAGE:latest .'
                    }
                }
            }
        }
    }

    stage('Push to Docker Hub') {
        steps {
            container('docker') {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKERHUB_USER/$APP_IMAGE:$TAG
                        docker push $DOCKERHUB_USER/$APP_IMAGE:latest
                        docker push $DOCKERHUB_USER/$DB_IMAGE:$TAG
                        docker push $DOCKERHUB_USER/$DB_IMAGE:latest
                    '''
                }
            }
        }
    }

 stage('Deploy to Kubernetes') {
         steps {
            container('kubectl') {
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
                        '''
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
