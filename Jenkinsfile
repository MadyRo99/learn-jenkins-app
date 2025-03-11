pipeline {
    agent any

    environment {
        BUILD_FILE_NAME = 'laptop.txt'
    }

    stages {
        stage('w/o docker') {
            steps {
                sh 'echo "Without Docker"'
            }
        }
        stage('w/ docker') {
            agent {
                docker {
                    image 'node:18-alpine'
                    reuseNode true
                }
            }
            steps {
                sh 'echo "With Docker"'
                sh 'npm --version'
            }
        }
    }
}
