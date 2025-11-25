pipeline {
    agent any

    triggers {
        githubPush()
    }

    stages {
        stage('git ') {
            steps {
                git branch: 'master', url :'https://github.com/LeithGharbi1/leith_gharbi_SIM1'
            }
        }
        stage('compilation') {
            steps {
                sh 'mvn clean compile'
            }
        }
    }
}
