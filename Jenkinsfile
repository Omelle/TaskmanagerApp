pipeline {
    agent any
    tools {
        sonarQube 'SonarQube Scanner' // Assurez-vous d'utiliser le nom de l'outil configuré dans Jenkins
    }
    stages {
        stage('Build') {
            steps {
                // Exécution de votre build (par exemple Maven, Gradle, etc.)
                sh 'mvn clean install'  // Si vous utilisez Maven
            }
        }
        stage('SonarQube analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQube Scanner'  // Utilisez le nom de votre configuration SonarQube Scanner
                    withSonarQubeEnv('SonarQube') {  // Utilisez le nom de votre serveur SonarQube
                        sh "${scannerHome}/bin/sonar-scanner"
                    }
                }
            }
        }
    }
    post {
        always {
            // Assurez-vous de récupérer les résultats de l'analyse dans un post-steps
            script {
                waitForQualityGate abortPipeline: true
            }
        }
    }
}
