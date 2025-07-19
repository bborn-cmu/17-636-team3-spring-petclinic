pipeline {
    agent any

    stages {
        stage('Build') {
            // The build server's agent has maven on it
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
