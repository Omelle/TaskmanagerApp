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
                timeout(time: 20, unit: 'MINUTES') // Augmenté à 20 minutes
            }
            steps {
                withSonarQubeEnv('MonServeurSonar') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        script {
                            sh '''#!/bin/bash
                                export SONAR_TOKEN=$SONAR_TOKEN
                                ${SONAR_SCANNER}/bin/sonar-scanner
                            '''
                        }
                    }
                }
            }
        }

        stage('Construire l’image Docker') {
            when {
                expression {
                    fileExists('report-task.txt') // Ne construire que si l'analyse Sonar a fonctionné
                }
            }
            steps {
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${BUILD_NUMBER} -f backend/Dockerfile .
                    """
                }
            }
        }

        stage('Pousser sur Docker Hub') {
            when {
                expression {
                    fileExists('report-task.txt')
                }
            }
            steps {
                script {
                    withDockerRegistry([credentialsId: "${DOCKER_CREDENTIALS}", url: ""]) {
                        sh "docker push ${DOCKER_IMAGE}:${BUILD_NUMBER}"
                    }
                }
            }
        }

        stage('Vérification Quality Gate') {
            when {
                expression {
                    fileExists('report-task.txt')
                }
            }
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
