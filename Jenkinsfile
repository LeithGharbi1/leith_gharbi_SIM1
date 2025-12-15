pipeline {
    agent any

    triggers {
        githubPush()  // Trigger pipeline on GitHub push
    }

    environment {
        DOCKERHUB_USER = 'leithgh'
        IMAGE_NAME = 'student-test'
        K8S_NAMESPACE = 'devops'
        K8S_MANIFEST_DIR = "${WORKSPACE}/k8s"   // full path in workspace
        MYSQL_DEPLOYMENT_NAME = 'mysql-deployment'
        SPRING_DEPLOYMENT_NAME = 'spring-deployment'
        KUBECONFIG_PATH = '/var/lib/jenkins/kubeconfig'  // local kubeconfig
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
                    KUBECONFIG=${KUBECONFIG_PATH} kubectl apply -f ${K8S_MANIFEST_DIR}/ -n ${K8S_NAMESPACE}
                    KUBECONFIG=${KUBECONFIG_PATH} kubectl rollout status deployment/${MYSQL_DEPLOYMENT_NAME} -n ${K8S_NAMESPACE} --timeout=180s
                    KUBECONFIG=${KUBECONFIG_PATH} kubectl rollout status deployment/${SPRING_DEPLOYMENT_NAME} -n ${K8S_NAMESPACE} --timeout=180s
                    KUBECONFIG=${KUBECONFIG_PATH} kubectl get pods -n ${K8S_NAMESPACE} -o wide
                    KUBECONFIG=${KUBECONFIG_PATH} kubectl get svc -n ${K8S_NAMESPACE}
                """
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
