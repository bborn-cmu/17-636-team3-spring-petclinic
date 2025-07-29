pipeline {
    agent any
    // for consistent caching of maven during subsequent builds on the same server
    environment {
      JAVA_TOOL_OPTIONS = '-Duser.home=/home/jenkins'
    }
    stages {
        stage('Build-Jar') {
            // The build server's agent has maven on it
            steps {
                sh 'mvn clean package -DskipTests'
                stash includes: 'target/*.jar', name: 'app-artifact'
            }
        }
        stage('Build-Image') {
            // The build server's agent has maven on it
            steps {
                // The "real" repo builds: 'docker.io/library/spring-petclinic:3.5.0-SNAPSHOT'
                // sh "./mvnw spring-boot:build-image -Dmodule.image.name=ghcr.io/bborn-cmu/17-636-team3-spring-petclinic:3.5.0-SNAPSHOT-build-${env.BUILD_NUMBER}"
                sh './mvnw spring-boot:build-image'
            }
        }
        stage('SonarQube Scan') {
            // the build server has the sonarqube CLI and configured sonarqube service
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh "mvn sonar:sonar -Dsonar.projectKey=spring-petclinic"
                }
            }
        }
        stage('ZAP Scan') {
            steps {
                script {
                    def appContainerName = "spring-pet-clinic-${env.BUILD_ID}"
                    def imageName = "docker.io/library/spring-petclinic:3.5.0-SNAPSHOT"

                    try {
                        // The system has a network just for inter-container communication during pipelines called jenkins-ci
                        sh "docker run -d --rm --name ${appContainerName} --network jenkins-ci ${imageName}"

                        // Run ZAP, the build system has a custom zap client script to make this easier in the /bin dir
                        sh "python /bin/zap_client.py --target \"http://${appContainerName}:8080\" --build ${env.BUILD_ID}"

                        // TODO: need to publish the HTML reports and remove them from the agent
                    } finally {
                        sh "docker rm -f ${appContainerName}"
                    }
                }
                
                archiveArtifacts artifacts: "zap_report_build_${env.BUILD_ID}.html", fingerprint: true
                publishHTML(target: [
                    reportFiles: "zap_report_build_${env.BUILD_ID}.html",
                    reportName: 'ZAP Scan Report'
                ])
            }
        }
        // TODO: need to configure a registry and auth info
        // stage('Push Image') {
        //     steps {
        //         docker.withRegistry('https://ghcr.io', 'your-credentials-id') {
        //             docker.image('ghcr.io/bborn-cmu/17-636-team3-spring-petclinic:3.5.0-SNAPSHOT-build-${env.BUILD_NUMBER}').push()
        //         }
        //     }
        // }
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
