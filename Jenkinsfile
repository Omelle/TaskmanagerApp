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
            options {
                timeout(time: 10, unit: 'MINUTES') // Timeout généreux pour éviter les blocages
            }
            steps {
                withSonarQubeEnv('MonServeurSonar') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        script {
                            // Exécuter l’analyse Sonar avec la variable d’environnement SONAR_TOKEN
                            sh """
                                export SONAR_TOKEN=${SONAR_TOKEN}
                                ${SONAR_SCANNER}/bin/sonar-scanner
                            """
                        }
                    }
                }
            }
        }

        stage('Construire l’image Docker') {
            steps {
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} -f backend/Dockerfile .
                    """
                }
            }
        }

        stage('Pousser sur Docker Hub') {
            steps {
                script {
                    withDockerRegistry([credentialsId: "${DOCKER_CREDENTIALS}", url: ""]) {
                        sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Vérification Quality Gate') {
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
