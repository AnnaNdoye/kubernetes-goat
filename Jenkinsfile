pipeline {
    agent any

    environment {
        // Remplacez par votre adresse email
        NOTIFICATION_EMAIL = "ndoyeanna754@gmail.com"
        REPO_URL = "${env.GIT_URL}"
    }

    stages {
        stage('Clonage du projet') {
            steps {
                // Jenkins récupère automatiquement le code depuis GitHub si configuré en mode SCM
                checkout scm
                echo "Projet récupéré avec succès."
            }
        }

        stage('Scan SAST (Semgrep)') {
            steps {
                echo "Lancement du scan SAST avec Semgrep..."
                // On utilise Docker pour lancer Semgrep sans l'installer sur la machine
                sh '''
                    docker run --rm -v "$(pwd):/src" returntocorp/semgrep semgrep scan \
                    --config=auto --json --output=semgrep-report.json || true
                '''
            }
        }

        stage('Scan IaC (Checkov)') {
            steps {
                echo "Lancement du scan IaC avec Checkov..."
                // On utilise Docker pour lancer Checkov sur les fichiers Kubernetes (YAML)
                sh '''
                    docker run --rm -v "$(pwd):/tf" bridgecrew/checkov -d /tf \
                    --framework kubernetes --output cli --output json > checkov-report.json || true
                '''
            }
        }

        stage('Rapport de Liaison VM & Résultats') {
            steps {
                script {
                    // Création d'un mini rapport textuel à intégrer dans le mail
                    def summary = """
                    ==================================================
                    RAPPORT AUDIT DE SÉCURITÉ - PIPELINE JENKINS
                    ==================================================
                    Dépôt Git analysé : ${REPO_URL}
                     Jenkins de build : ${env.BUILD_URL}
                    
                    Résultats des scans :
                    - Scan SAST (Semgrep) effectué.
                    - Scan IaC (Checkov) effectué.
                    
                    Les fichiers JSON complets (semgrep-report.json et checkov-report.json) 
                    sont archivés sur le serveur Jenkins.
                    ==================================================
                    """
                    writeFile file: 'summary.txt', text: summary
                    echo summary
                }
                // Archive les rapports dans Jenkins pour consultation future
                archiveArtifacts artifacts: '*-report.json, summary.txt', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            // Envoi du mail avec le résumé des tests
            mail to: "${env.NOTIFICATION_EMAIL}",
                 subject: "Rapport de sécurité Jenkins - Build #${env.BUILD_NUMBER}",
                 body: readFile('summary.txt'),
                 mimeType: 'text/plain'
        }
    }
}
