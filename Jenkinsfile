pipeline {
    agent any

    environment {
        PROJECT_NAME   = "ZeroTrace"
        RECIPIENT_MAIL = "mameaicha185@gmail.com"
        PYTHON         = "python3"
        PIP            = "pip3"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/mameaicha08/ZeroTrace.git'
            }
        }

        stage('Setup Environnement') {
            steps {
                sh '''
                    pip3 install --upgrade pip
                    pip3 install -r requirements.txt
                    pip3 install bandit safety flake8
                '''
            }
            post {
                failure {
                    emailext(
                        subject: "❌ [ZeroTrace] SETUP ÉCHOUÉ - Build #${BUILD_NUMBER}",
                        body: """<h2>Échec installation dépendances</h2>
                                 <p><a href="${BUILD_URL}console">Voir les logs</a></p>""",
                        mimeType: 'text/html',
                        to: "${RECIPIENT_MAIL}"
                    )
                }
            }
        }

        stage('Build - Vérification syntaxe') {
            steps {
                sh '''
                    python3 -m py_compile main.py
                    python3 -m py_compile crypto_utils.py
                    python3 -m py_compile signalement.py
                    python3 -m py_compile auditeur.py
                    python3 -m py_compile audit.py
                    echo "✅ Syntaxe OK"
                '''
            }
            post {
                failure {
                    emailext(
                        subject: "❌ [ZeroTrace] ERREUR SYNTAXE - Build #${BUILD_NUMBER}",
                        body: """<h2>Erreur de syntaxe Python détectée</h2>
                                 <p><a href="${BUILD_URL}console">Voir les logs</a></p>""",
                        mimeType: 'text/html',
                        to: "${RECIPIENT_MAIL}"
                    )
                }
            }
        }

        stage('Lint - Flake8') {
            steps {
                sh '''
                    mkdir -p reports
                    flake8 . --max-line-length=120 \
                             --exclude=__pycache__,.git \
                             --output-file=reports/flake8-report.txt \
                             || true
                    cat reports/flake8-report.txt || echo "Aucune erreur"
                '''
            }
        }

        stage('SAST - Bandit') {
            steps {
                sh '''
                    mkdir -p reports
                    bandit -r . --exclude ./.git,__pycache__ \
                           -f html -o reports/bandit-report.html -ll || true
                    bandit -r . --exclude ./.git,__pycache__ \
                           -f txt  -o reports/bandit-report.txt  -ll || true
                    cat reports/bandit-report.txt || true
                '''
            }
            post {
                always {
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'reports',
                        reportFiles: 'bandit-report.html',
                        reportName: 'Rapport SAST - Bandit'
                    ])
                }
            }
        }

        stage('SCA - Safety') {
            steps {
                sh '''
                    mkdir -p reports
                    safety check --file=requirements.txt \
                                 --output text \
                                 > reports/safety-report.txt || true
                    cat reports/safety-report.txt || true
                '''
            }
            post {
                always {
                    publishHTML([
                        allowMissing: true,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'reports',
                        reportFiles: 'safety-report.txt',
                        reportName: 'Rapport SCA - Safety'
                    ])
                }
            }
        }
    }

    post {
        success {
            emailext(
                subject: "✅ [ZeroTrace] Pipeline RÉUSSI - Build #${BUILD_NUMBER}",
                body: """
                <html><body>
                <h2 style="color:green;">✅ Pipeline ZeroTrace — Succès</h2>
                <table border="1" cellpadding="8">
                  <tr><td><b>Projet</b></td><td>${PROJECT_NAME}</td></tr>
                  <tr><td><b>Build #</b></td><td>${BUILD_NUMBER}</td></tr>
                  <tr><td><b>Durée</b></td><td>${currentBuild.durationString}</td></tr>
                </table>
                <h3>📊 Rapports :</h3>
                <ul>
                  <li><a href="${BUILD_URL}Rapport_SAST_-_Bandit">Rapport SAST Bandit</a></li>
                  <li><a href="${BUILD_URL}Rapport_SCA_-_Safety">Rapport SCA Safety</a></li>
                  <li><a href="${BUILD_URL}console">Logs complets</a></li>
                </ul>
                </body></html>
                """,
                mimeType: 'text/html',
                to: "${RECIPIENT_MAIL}",
                attachmentsPattern: 'reports/bandit-report.html,reports/safety-report.txt'
            )
        }

        failure {
            emailext(
                subject: "❌ [ZeroTrace] Pipeline ÉCHOUÉ - Build #${BUILD_NUMBER}",
                body: """
                <html><body>
                <h2 style="color:red;">❌ Pipeline ZeroTrace — Échec</h2>
                <table border="1" cellpadding="8">
                  <tr><td><b>Build #</b></td><td>${BUILD_NUMBER}</td></tr>
                  <tr><td><b>Statut</b></td><td style="color:red;">FAILED</td></tr>
                </table>
                <p><a href="${BUILD_URL}console"><b>👉 Voir les logs complets</b></a></p>
                </body></html>
                """,
                mimeType: 'text/html',
                to: "${RECIPIENT_MAIL}",
                attachmentsPattern: 'reports/*.txt,reports/*.html'
            )
        }

        unstable {
            emailext(
                subject: "⚠️ [ZeroTrace] Vulnérabilités détectées - Build #${BUILD_NUMBER}",
                body: """
                <html><body>
                <h2 style="color:orange;">⚠️ Vulnérabilités de sécurité détectées</h2>
                <ul>
                  <li><a href="${BUILD_URL}Rapport_SAST_-_Bandit">Rapport SAST Bandit</a></li>
                  <li><a href="${BUILD_URL}Rapport_SCA_-_Safety">Rapport SCA Safety</a></li>
                </ul>
                </body></html>
                """,
                mimeType: 'text/html',
                to: "${RECIPIENT_MAIL}",
                attachmentsPattern: 'reports/*.txt,reports/*.html'
            )
        }
    }
}
