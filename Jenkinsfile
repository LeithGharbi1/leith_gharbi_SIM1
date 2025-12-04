pipeline {
    agent any

    triggers {
        githubPush()
    }

    environment {
        DOCKERHUB_USER = 'leithgh'            // 👉 change to your Docker Hub username
        IMAGE_NAME = 'student-test'    // 👉 change if needed
    }

    stages {

        stage('Git Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/LeithGharbi1/leith_gharbi_SIM1'
            }
        }

        stage('Maven Compile') {
            steps {
                sh 'mvn compile'
            }
        }

        

        stage('Maven Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

  stage('MVN SonarQube') {
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh """
                        mvn sonar:sonar \
                        -Dsonar.projectKey=student-test \
                        -Dsonar.host.url=http://localhost:9000 \
                        -Dsonar.login=admin
                    """
                }
            }
        }
        
       stage('Build Docker Image') {
    steps {
        dir("${WORKSPACE}") {   
            sh """
                docker build -t ${DOCKERHUB_USER}/${IMAGE_NAME}:latest .
            """
        }
    }
}


        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                                 usernameVariable: 'USER',
                                                 passwordVariable: 'PASS')]) {
                    sh """
                        echo "$PASS" | docker login -u "$USER" --password-stdin
                    """
                }
            }
        }

        stage('Docker Push') {
            steps {
                sh "docker push ${DOCKERHUB_USER}/${IMAGE_NAME}:latest"
            }
        }
    }
}
