pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'omaelle/tasmanagerapp'
        DOCKER_CREDENTIALS = 'credential-dockerhub'
        SONAR_SCANNER = tool name: 'SonarScanner', type: 'hudson.plugins.sonar.SonarRunnerInstallation'
    }

    stages {
        stage('Cloner le code') {
            steps {
                git branch: 'main', url: 'https://github.com/Omelle/TaskmanagerApp.git'
            }
        }

        stage('Analyse SonarQube') {
            steps {
                withSonarQubeEnv('MonServeurSonar') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh "${SONAR_SCANNER}/bin/sonar-scanner -Dsonar.login=$SONAR_TOKEN"
                    }
                }
            }
        }

        stage('Construire l’image Docker') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER -f backend/Dockerfile .'
                }
            }
        }

        stage('Pousser sur Docker Hub') {
            steps {
                script {
                    withDockerRegistry([credentialsId: "$DOCKER_CREDENTIALS", url: ""]) {
                        sh 'docker push $DOCKER_IMAGE:$BUILD_NUMBER'
                    }
                }
            }
        }

        // Optionnel : ajout du Quality Gate
        stage('Vérification Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
