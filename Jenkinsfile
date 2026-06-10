pipeline {
    agent any

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
                // Utilisation du volume nommé Jenkins pour que Docker retrouve le bon chemin sous Windows/WSL
                sh '''
                    docker run --rm -v jenkins_home:/var/jenkins_home -w /var/jenkins_home/workspace/Audit-Securite-K8s returntocorp/semgrep semgrep scan --config=auto --json --output=semgrep-report.json || true
                '''
            }
        }

        stage('Scan IaC (Checkov)') {
            steps {
                echo "Lancement du scan IaC avec Checkov..."
                // Même logique : on monte le volume nommé global 'jenkins_home'
                sh '''
                    docker run --rm -v jenkins_home:/var/jenkins_home -w /var/jenkins_home/workspace/Audit-Securite-K8s bridgecrew/checkov -d . --framework kubernetes --output json > checkov-report.json || true
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
                    - Scan SAST (Semgrep) exécuté sur le volume partagé.
                    - Scan IaC (Checkov) exécuté sur le volume partagé.
                    
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
            mail to: "${env.NOTIFICATION_EMAIL}",
                 subject: "Rapport de sécurité Jenkins - Build #${env.BUILD_NUMBER}",
                 body: readFile('summary.txt'),
                 mimeType: 'text/plain'
        }
    }
}
