pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'omaelle/tasmanagerapp:latest'
    }

    stages {
        stage('Cloner le code') {
            steps {
                git branch: 'main', url: 'https://github.com/Omelle/TaskmanagerApp.git'
            }
        }

        stage('Tests & couverture') {
            steps {
                dir('backend') {
                    sh 'mkdir -p ./test-results'
                    sh 'npm install'
                    sh 'npx mocha ./test/basic.test.js --reporter mocha-junit-reporter --reporter-options mochaFile=./test-results/results.xml'
                }
            }
        }

        stage('Analyse SonarQube') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                echo 'Analyse SonarQube ici...'
            }
        }

        stage('Construire l’image Docker') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                sh 'docker build -t $DOCKER_IMAGE .'
            }
        }

        stage('Pousser sur Docker Hub') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'credential-dockerhub', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $DOCKER_IMAGE
                    '''
                }
            }
        }
    }

    post {
        always {
            junit 'backend/test-results/results.xml'
        }
    }
}
