pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        DOCKERHUB_USER = 'leithgh'
        IMAGE_NAME = 'student-test'
        K8S_NAMESPACE = 'devops'
        K8S_MANIFEST_DIR = '/var/lib/jenkins/workspace/Leith Gharbi 4 SIM1/k8s'
        MYSQL_DEPLOYMENT_NAME = 'mysql-deployment'
        SPRING_DEPLOYMENT_NAME = 'spring-deployment'
        KUBECONFIG_PATH = '/var/lib/jenkins/kubeconfig'
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/LeithGharbi1/leith_gharbi_SIM1'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh "docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest ."

                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'USER',
                                                 passwordVariable: 'PASS')]) {
                    sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
                }

                sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
            }
        }

        stage('Kubernetes Deploy') {
            steps {
                sh """
                    export KUBECONFIG=${KUBECONFIG_PATH}
                    kubectl apply -f "${K8S_MANIFEST_DIR}/mysql-deployment.yaml" -n ${K8S_NAMESPACE}
                    kubectl apply -f "${K8S_MANIFEST_DIR}/spring-deployment.yaml" -n ${K8S_NAMESPACE}
                """
            }
        }

        stage('Verify Rollout') {
            steps {
                sh """
                    export KUBECONFIG=${KUBECONFIG_PATH}
                    echo 'Waiting for MySQL rollout...'
                    kubectl -n ${K8S_NAMESPACE} rollout status deployment/${MYSQL_DEPLOYMENT_NAME} --timeout=180s
                    echo 'Waiting for Spring Boot rollout...'
                    kubectl -n ${K8S_NAMESPACE} rollout status deployment/${SPRING_DEPLOYMENT_NAME} --timeout=180s

                    echo 'Pods and Services:'
                    kubectl -n ${K8S_NAMESPACE} get pods -o wide
                    kubectl -n ${K8S_NAMESPACE} get svc
                """
            }
        }

    }

    post {
        always { echo 'Pipeline finished' }
        success { echo 'Pipeline succeeded!' }
        failure { echo 'Pipeline failed!' }
    }
}
