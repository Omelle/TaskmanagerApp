pipeline {
    agent any

    environment {
        // Ajoute ici des variables d’environnement si besoin
    }

    stages {
        stage('Cloner le code') {
            steps {
                git 'https://github.com/Omelle/TaskmanagerApp.git'
            }
        }

        stage('Tests & couverture') {
            steps {
                dir('backend') {
                    // Création du dossier pour les résultats de test
                    sh 'mkdir -p ./test-results'
                    
                    // Installation des dépendances
                    sh 'npm install'
                    
                    // Exécution des tests avec le reporter JUnit
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
                echo 'Construction Docker ici...'
            }
        }

        stage('Pousser sur Docker Hub') {
            when {
                expression { currentBuild.resultIsBetterOrEqualTo('SUCCESS') }
            }
            steps {
                echo 'Push vers Docker Hub ici...'
            }
        }
    }

    post {
        always {
            junit 'backend/test-results/results.xml'
        }
    }
}
