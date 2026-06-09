pipeline {
    agent any

    environment {
        PROJECT_NAME   = "ZeroTrace"
        RECIPIENT_MAIL = "mameaicha185@gmail.com"
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
                bat '''
                    pip install --upgrade pip
                    pip install -r requirements.txt
                    pip install bandit safety flake8
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

        stage('Build - Verification syntaxe') {
            steps {
                bat '''
                    python -m py_compile main.py
                    python -m py_compile crypto_utils.py
                    python -m py_compile signalement.py
                    python -m py_compile auditeur.py
                    python -m py_compile audit.py
                    echo Syntaxe OK
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
                bat '''
                    if not exist reports mkdir reports
                    flake8 . --max-line-length=120 --exclude=__pycache__,.git --output-file=reports/flake8-report.txt || exit /b 0
                    type reports\\flake8-report.txt
                '''
            }
        }

        stage('SAST - Bandit') {
            steps {
                bat '''
                    if not exist reports mkdir reports
                    bandit -r . --exclude ./.git,__pycache__ -f html -o reports/bandit-report.html -ll || exit /b 0
                    bandit -r . --exclude ./.git,__pycache__ -f txt  -o reports/bandit-report.txt  -ll || exit /b 0
                    type reports\\bandit-report.txt || exit /b 0
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
                bat '''
                    if not exist reports mkdir reports
                    safety check --file=requirements.txt --output text > reports/safety-report.txt || exit /b 0
                    type reports\\safety-report.txt || exit /b 0
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
                subject: "✅ [ZeroTrace] Pipeline REUSSI - Build #${BUILD_NUMBER}",
                body: """
                <html><body>
                <h2 style="color:green;">Pipeline ZeroTrace - Succes</h2>
                <table border="1" cellpadding="8">
                  <tr><td><b>Projet</b></td><td>${PROJECT_NAME}</td></tr>
                  <tr><td><b>Build</b></td><td>#${BUILD_NUMBER}</td></tr>
                  <tr><td><b>Duree</b></td><td>${currentBuild.durationString}</td></tr>
                </table>
                <h3>Rapports :</h3>
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
                subject: "❌ [ZeroTrace] Pipeline ECHOUE - Build #${BUILD_NUMBER}",
                body: """
                <html><body>
                <h2 style="color:red;">Pipeline ZeroTrace - Echec</h2>
                <table border="1" cellpadding="8">
                  <tr><td><b>Build</b></td><td>#${BUILD_NUMBER}</td></tr>
                  <tr><td><b>Statut</b></td><td style="color:red;">FAILED</td></tr>
                </table>
                <p><a href="${BUILD_URL}console">Voir les logs complets</a></p>
                </body></html>
                """,
                mimeType: 'text/html',
                to: "${RECIPIENT_MAIL}",
                attachmentsPattern: 'reports/*.txt,reports/*.html'
            )
        }

        unstable {
            emailext(
                subject: "⚠️ [ZeroTrace] Vulnerabilites detectees - Build #${BUILD_NUMBER}",
                body: """
                <html><body>
                <h2 style="color:orange;">Vulnerabilites de securite detectees</h2>
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
