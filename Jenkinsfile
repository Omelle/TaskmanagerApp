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
                        script {
                            // Définir la commande SonarQube
                            def scannerCmd = "${SONAR_SCANNER}/bin/sonar-scanner -Dsonar.login=$SONAR_TOKEN"

                            // Tentative d'exécution du scanner
                            try {
                                sh scannerCmd
                            } catch (Exception e) {
                                // En cas d'échec, arrêter le pipeline et afficher l'erreur
                                currentBuild.result = 'FAILURE'
                                error "SonarQube analysis failed: ${e.getMessage()}"
                            }
                        }
                    }
                }
            }
        }

        stage('Construire l’image Docker') {
            steps {
                script {
                    // Construction de l'image Docker
                    sh 'docker build -t $DOCKER_IMAGE:$BUILD_NUMBER -f backend/Dockerfile .'
                }
            }
        }

        stage('Pousser sur Docker Hub') {
            steps {
                script {
                    // Pousser l'image Docker sur Docker Hub
                    withDockerRegistry([credentialsId: "$DOCKER_CREDENTIALS", url: ""]) {
                        sh 'docker push $DOCKER_IMAGE:$BUILD_NUMBER'
                    }
                }
            }
        }

        // Optionnel : ajout de la vérification du Quality Gate
        stage('Vérification Quality Gate') {
            steps {
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
