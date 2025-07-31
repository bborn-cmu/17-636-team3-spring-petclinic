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
                // sh "./mvnw spring-boot:build-image -DskipTests -Dmodule.image.name=ghcr.io/bborn-cmu/17-636-team3-spring-petclinic:3.5.0-SNAPSHOT-build-${env.BUILD_NUMBER}"
                sh './mvnw spring-boot:build-image -DskipTests'
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
                    //def imageName = "ghcr.io/bborn-cmu/17-636-team3-spring-petclinic:3.5.0-SNAPSHOT-build-${env.BUILD_NUMBER}"
                    try {
                        // The system has a network just for inter-container communication during pipelines called jenkins-ci
                        sh "docker run -d --rm --name ${appContainerName} --network jenkins-ci ${imageName}"
                        // Website needs about 18 seconds to start
                        sh "sleep 30"
                        // Run ZAP, the build system has a custom zap client script to make this easier in the /bin dir
                        sh "python /bin/zap_client.py --target \"http://${appContainerName}:8080\" --build ${env.BUILD_ID}"

                        // TODO: need to publish the HTML reports and remove them from the agent
                    } finally {
                        sh "docker rm -f ${appContainerName}"
                    }
                }
                
                archiveArtifacts artifacts: "zap_report_build_${env.BUILD_ID}.html", fingerprint: true
                publishHTML(target: [
                    reportDir: '.', 
                    reportFiles: "zap_report_build_${env.BUILD_ID}.html",
                    reportName: 'ZAP Scan Report'
                ])
            }
        }
        stage('Deploy to VM (via Ansible)') {
            steps {
                echo "Deploying Spring Petclinic to VM..."
                unstash 'app-artifact'
                sh 'mkdir -p ansible/files && cp target/*.jar ansible/files/'
                dir('ansible') {
                    sh 'ansible-playbook -i inventory.ini deploy.yml'
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
        success {
            echo "Pipeline completed successfully!"
            archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
        }
        failure {
            echo "Pipeline failed."
        }
    }
}
