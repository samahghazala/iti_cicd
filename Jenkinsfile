pipeline {
    agent any

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
                script {
                    dir('Docker-files/app') {
                        sh "docker build --build-arg BUILD_NUMBER=${env.BUILD_NUMBER} -t ${DOCKERHUB_USER}/${APP_IMAGE}:${TAG} ."
                    }
                }
            }
        }

        stage('Build DB Image') {
            steps {
                script {
                    dir('Docker-files/db') {
                        sh "docker build -t ${DOCKERHUB_USER}/${DB_IMAGE}:${TAG} ."
                    }
                }
            }
        }

        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                        echo $PASS | docker login -u $USER --password-stdin
                        docker push ${DOCKERHUB_USER}/${APP_IMAGE}:${TAG}
                        docker push ${DOCKERHUB_USER}/${DB_IMAGE}:${TAG}
                        docker logout
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh '''
                    # Apply only Kubernetes YAML files
                    kubectl apply -f app-secret.yml
                    kubectl apply -f db-CIP.yml
                    kubectl apply -f mc-CIP.yml
                    kubectl apply -f mcdep.yml
                    kubectl apply -f rmq-CIP-service.yml
                    kubectl apply -f rmq-dep.yml
                    kubectl apply -f vproapp-service.yml
                    kubectl apply -f vproappdep.yml
                    kubectl apply -f vprodbdep.yml
                    # Force rollout restart to guarantee new pods run the updated image
                    kubectl rollout restart deployment/vproappdep || true
                    kubectl rollout restart deployment/vprodbdep || true
                '''
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

