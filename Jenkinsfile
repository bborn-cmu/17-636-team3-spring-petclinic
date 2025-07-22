pipeline {
    agent any
    // for consistent caching of maven during subsequent builds on the same server
    environment {
      JAVA_TOOL_OPTIONS = '-Duser.home=/home/jenkins'
    }
    stages {
        stage('Build') {
            // The build server's agent has maven on it
            steps {
                sh 'mvn clean package -DskipTests'
                stash includes: 'target/*.jar', name: 'app-artifact'
            }
        }
        // stage('Build-Image') {
        //     // The build server's agent has maven on it
        //     steps {
        //         sh './mvnw spring-boot:build-image'
        //     }
        // }
        stage('SonarQube Scan') {
            // the build server has the sonarqube CLI and configured sonarqube service
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar -Dsonar.projectKey=spring-petclinic"
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
        success {
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
    }
}
