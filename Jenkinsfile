pipeline {
    agent any // On réutilise l'agent classique

    environment {
        NOTIFICATION_EMAIL = "ndoyeanna754@gmail.com"
        REPO_URL = "https://github.com/AnnaNdoye/kubernetes-goat.git"
    }

    stages {
        stage('Clonage du projet') {
            steps {
                checkout scm
                echo "Projet récupéré avec succès."
            }
        }

        stage('Scan SAST (Semgrep)') {
            steps {
                echo "Lancement du scan SAST avec Semgrep..."
                // On utilise une image "docker" légère pour exécuter la commande à l'extérieur
                sh '''
                    docker run --rm -v "/var/jenkins_home/workspace/Audit-Securite-K8s:/src" returntocorp/semgrep semgrep scan --config=auto --json --output=semgrep-report.json || true
                '''
            }
        }

        stage('Scan IaC (Checkov)') {
            steps {
                echo "Lancement du scan IaC avec Checkov..."
                sh '''
                    docker run --rm -v "/var/jenkins_home/workspace/Audit-Securite-K8s:/tf" bridgecrew/checkov -d /tf --framework kubernetes --output json > checkov-report.json || true
                '''
            }
        }

        stage('Rapport & Archivage') {
            steps {
                script {
                    def summary = """
                    ==================================================
                    RAPPORT AUDIT DE SÉCURITÉ - PIPELINE JENKINS
                    ==================================================
                    Dépôt Git analysé : ${REPO_URL}
                    Jenkins de build : ${env.BUILD_URL}
                    
                    Résultats des scans :
                    - Scan SAST (Semgrep) terminé.
                    - Scan IaC (Checkov) terminé.
                    
                    Les fichiers JSON complets ont été archivés sur Jenkins.
                    ==================================================
                    """
                    writeFile file: 'summary.txt', text: summary
                    echo summary
                }
                archiveArtifacts artifacts: 'summary.txt, *report.json', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            // Envoi du mail automatique
            mail to: "${env.NOTIFICATION_EMAIL}",
                 subject: "Rapport de sécurité Jenkins - Build #${env.BUILD_NUMBER}",
                 body: readFile('summary.txt'),
                 mimeType: 'text/plain'
        }
    }
}
