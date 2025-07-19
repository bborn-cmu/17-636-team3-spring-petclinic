pipeline {
    agent any

    environment {
        MAVEN_IMAGE = 'maven:3.9.10-sapmachine-24'
    }

    stages {
        stage('Build') {
            agent {
                docker {
                    image "${MAVEN_IMAGE}"
                    args '-v /root/.m2:/root/.m2'
                }
            }
            steps {
                sh 'mvn clean package -DskipTests'
                stash includes: 'target/*.jar', name: 'app-artifact'
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}
