pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'omaelle/tasmanagerapp'
        DOCKER_CREDENTIALS = 'credential-dockerhub'
    }

    stages {
        stage('Cloner le code') {
            steps {
                git branch: 'main', url: 'https://github.com/Omelle/TaskmanagerApp.git'
            }
        }

        stage('Analyse SonarQube') {
            steps {
                withSonarQubeEnv('SonarQube') { // Assure-toi que ce nom correspond à ce que tu as configuré dans Jenkins
                    sh 'mvn clean verify sonar:sonar'
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

        // stage('Déployer sur Kubernetes') {
        //     steps {
        //         script {
        //             sh 'kubectl apply -f k8s/deployment.yaml'
        //             sh 'kubectl apply -f k8s/service.yaml'
        //         }
        //     }
        // }
    }
}
