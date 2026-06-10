pipeline {
    agent none 

    environment {
        NOTIFICATION_EMAIL = "ndoyeanna754@gmail.com"
        REPO_URL = "https://github.com/AnnaNdoye/kubernetes-goat.git"
    }

    stages {
        stage('Clonage du projet') {
            agent any
            steps {
                checkout scm
                echo "Projet récupéré avec succès."
            }
        }

        stage('Scan SAST (Semgrep)') {
            agent {
                docker { 
                    image 'returntocorp/semgrep'
                    args '-u 0'
                }
            }
            steps {
                echo "Lancement du scan SAST avec Semgrep..."
                sh 'semgrep scan --config=auto --json --output=semgrep-report.json || true'
            }
        }

        stage('Scan IaC (Checkov)') {
            agent {
                docker { 
                    image 'bridgecrew/checkov:latest'
                    args '-u 0'
                }
            }
            steps {
                echo "Lancement du scan IaC avec Checkov..."
                sh 'checkov -d . --framework kubernetes --output json > checkov-report.json || true'
            }
        }

        stage('Rapport & Archivage') {
            agent any
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
                    
                    Les fichiers JSON complets ont été archivés.
                    ==================================================
                    """
                    writeFile file: 'summary.txt', text: summary
                    echo summary
                }
                archiveArtifacts artifacts: '*-report.json, summary.txt', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            node('any') {
                // Envoi du mail automatique à ndoyeanna754@gmail.com
                mail to: "${env.NOTIFICATION_EMAIL}",
                     subject: "Rapport de sécurité Jenkins - Build #${env.BUILD_NUMBER}",
                     body: readFile('summary.txt'),
                     mimeType: 'text/plain'
            }
        }
    }
}
