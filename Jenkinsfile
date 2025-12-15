pipeline {
    agent any

    triggers {
        githubPush()  // Trigger pipeline on GitHub push
    }

    environment {
        DOCKERHUB_USER = 'leithgh'
        IMAGE_NAME = 'student-test'
        K8S_NAMESPACE = 'devops'
        K8S_MANIFEST_DIR = 'k8s'            
        MYSQL_DEPLOYMENT_NAME = 'mysql-deployment'
        SPRING_DEPLOYMENT_NAME = 'spring-deployment'
        KUBECONFIG_CREDENTIAL_ID = 'kubeconfig'
    }

    stages {

        // 1️⃣ Git Checkout
        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/LeithGharbi1/leith_gharbi_SIM1'
            }
        }

        // 2️⃣ Build (Maven compile + package)
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        // 3️⃣ Docker Build & Push
        stage('Docker Build & Push') {
            steps {
                // Build image
                sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest ."

                // Login to Docker Hub
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'USER',
                                                 passwordVariable: 'PASS')]) {
                    sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
                }

                // Push image
                sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
            }
        }

        // 4️⃣ Kubernetes Deploy (apply manifests)
        stage('Kubernetes Deploy') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIAL_ID}", variable: 'KUBECONFIG')]) {
                    sh """
                        kubectl apply -f ${K8S_MANIFEST_DIR}/mysql-deployment.yaml -n ${K8S_NAMESPACE}
                        kubectl apply -f ${K8S_MANIFEST_DIR}/spring-deployment.yaml -n ${K8S_NAMESPACE}
                    """
                }
            }
        }

        // 5️⃣ Deploy MySQL & Spring Boot on K8s (verify rollout)
        stage('Deploy MySQL & Spring Boot on K8s') {
            steps {
                withCredentials([file(credentialsId: "${KUBECONFIG_CREDENTIAL_ID}", variable: 'KUBECONFIG')]) {
                    sh """
                        echo 'Waiting for MySQL rollout...'
                        kubectl -n ${K8S_NAMESPACE} rollout status deployment/${MYSQL_DEPLOYMENT_NAME} --timeout=180s
                        echo 'Waiting for Spring Boot rollout...'
                        kubectl -n ${K8S_NAMESPACE} rollout status deployment/${SPRING_DEPLOYMENT_NAME} --timeout=180s

                        echo 'Checking Pods and Services'
                        kubectl -n ${K8S_NAMESPACE} get pods -o wide
                        kubectl -n ${K8S_NAMESPACE} get svc
                    """
                }
            }
        }

    }

    post {
        always {
            echo 'Pipeline finished'
        }
        success {
            echo 'Pipeline succeeded!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
