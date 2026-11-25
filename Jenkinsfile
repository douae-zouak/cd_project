pipeline {
    agent any

    environment {
        NEXUS_URL = 'http://localhost:8081/repository/python-release/Release.zip'
    }

    stages {
        stage('Nettoyage du Workspace') {
            steps {
                deleteDir()
            }
        }

        stage('Téléchargement de la Release') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'nexus-creds',
                                                  usernameVariable: 'NEXUS_USER',
                                                  passwordVariable: 'NEXUS_PASS')]) {
                    bat '''
                    curl --fail -u %NEXUS_USER%:%NEXUS_PASS% ^
                    -o Release.zip ^
                    %NEXUS_URL%
                    '''
                }
            }
        }

        stage('Vérification du téléchargement') {
            steps {
                bat '''
                if not exist Release.zip (
                    echo ERREUR : Release.zip introuvable
                    exit /b 1
                )
                dir
                '''
            }
        }

        stage('Décompression de la Release') {
            steps {
                bat '''
                powershell -Command "Expand-Archive -Path Release.zip -DestinationPath Release -Force"
                '''
            }
        }

        stage('Vérification de la Release') {
            steps {
                bat '''
                if not exist Release (
                    echo ERREUR : dossier Release introuvable
                    exit /b 1
                )
                dir Release
                '''
            }
        }

        stage('Déploiement DEV') {
            steps {
                bat '''
                if not exist C:\\DEV mkdir C:\\DEV
                xcopy Release C:\\DEV /E /I /Y
                '''
            }
        }

        stage('Vérification DEV') {
            steps {
                bat '''
                echo ===== CONTENU DEV =====
                dir C:\\DEV
                if exist C:\\DEV\\calculator.py (
                    python C:\\DEV\\calculator.py
                )
                '''
            }
        }

        stage('Déploiement TEST') {
            steps {
                bat '''
                if not exist C:\\TEST mkdir C:\\TEST
                xcopy Release C:\\TEST /E /I /Y
                '''
            }
        }

        stage('Tests fonctionnels') {
            steps {
                bat '''
                if exist C:\\TEST\\test_calculator.py (
                    python C:\\TEST\\test_calculator.py
                ) else (
                    echo Aucun test fonctionnel trouve.
                )
                '''
            }
        }
    }

    post {
        success {
            echo '======================================'
            echo 'Déploiement terminé avec succès.'
            echo '======================================'
        }
        failure {
            echo '======================================'
            echo 'Le pipeline a échoué.'
            echo 'Vérifier les logs Jenkins.'
            echo '======================================'
        }
        always {
            archiveArtifacts artifacts: 'Release.zip', fingerprint: true
        }
    }
}