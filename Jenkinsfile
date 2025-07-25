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
                    // Start the container just built
                    def appContainerId = sh(
                        // script: "docker run -d -p 8080:8080 ghcr.io/bborn-cmu/17-636-team3-spring-petclinic:3.5.0-SNAPSHOT-build-${env.BUILD_NUMBER}",
                        script: "docker run -d -p 9002:8080 docker.io/library/spring-petclinic:3.5.0-SNAPSHOT",
                        returnStdout: true
                    ).trim()
                    // takes a few seconds for the website to render
                    sleep 30

                    // Run ZAP 
                    // TODO: figure out the ZAP plugin to run its scan here on the running image (and use a custom network here vs hard coded ports)
                    sh '''
                    curl -klsS -vvv http://localhost:9002
                    '''

                    // Stop the app
                    sh "docker stop ${appContainerId}"
                }
                script {
                    def appContainerName = "spring-pet-clinic-${env.BUILD_ID}"
                    def imageName = "docker.io/library/spring-petclinic:3.5.0-SNAPSHOT"

                    try {
                        // The system has a network just for inter-container communication during pipelines called jenkins-ci
                        sh "docker run -d --rm --name ${appContainerName} --network jenkins-ci ${imageName}"

                        // Run ZAP, the build system has a custom zap client script to make this easier in the /bin dir
                        sh "zap_client.sh http://${appContainerName}:8080"

                        // TODO: need to publish the HTML reports
                    } finally {
                        sh "docker rm -f ${appContainerName}"
                    }
                }
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
