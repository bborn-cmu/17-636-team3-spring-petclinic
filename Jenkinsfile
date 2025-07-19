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

        stage('SonarQube Scan') {
            // the build server has the sonarqube CLI and configured sonarqube service
            steps {
                withSonarQubeEnv('SonarQube') {
                sh "mvn sonar:sonar -Dsonar.projectKey=${SONAR_PROJECT_KEY}"
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
    }
}
