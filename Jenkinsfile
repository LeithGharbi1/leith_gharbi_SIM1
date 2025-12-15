pipeline {
    agent any

    triggers {
        githubPush()
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


 // stage('MVN SonarQube') {
           // steps {
              //  withSonarQubeEnv('sonarqube') {
                  //  sh """
                      //  mvn sonar:sonar \
                      //  -Dsonar.projectKey=student-test \
                       // -Dsonar.host.url=http://192.168.33.10:9000 \
                     //   -Dsonar.login=admin
               //     """
               // }
           // }
      //  }
        
       stage('Build Docker Image') {
    steps {
        dir("${WORKSPACE}") {   
            sh """
                docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest .
            """
        }
    }
}


 stage('Docker Build & Push') {
            steps {
                // Build Docker image
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
        stage('Kubernetes Deploy') {
            steps {
                // Apply MySQL first
                sh "kubectl apply -f mysql-deployment.yaml -n ${K8S_NAMESPACE}"

                // Apply Spring Boot
                sh "kubectl apply -f spring-deployment.yaml -n ${K8S_NAMESPACE}"
            }
        }

        stage('Deploy MySQL & Spring Boot on K8s') {
            steps {
                echo "Verifying Pods and Services..."

                sh "kubectl get pods -n ${K8S_NAMESPACE}"
                sh "kubectl get svc -n ${K8S_NAMESPACE}"

                // Optional: tail logs of Spring Boot Pod
                sh """
                SPRING_POD=\$(kubectl get pods -n ${K8S_NAMESPACE} -l app=spring-app -o jsonpath='{.items[0].metadata.name}')
                kubectl logs \$SPRING_POD -n ${K8S_NAMESPACE} --tail=50
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
