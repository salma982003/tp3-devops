pipeline {
    agent any
    options { 
        timestamps() 
    }
    
    environment {
        IMAGE = 'votre_nom_d_utilisateur/monapp'
        TAG = "build-${env.BUILD_NUMBER}"
    }
    
    stages {
        stage('Checkout Git') {
            steps {
                checkout scm
                bat 'echo "Repository cloné avec succès"'
                bat 'dir'
            }
        }
        
        stage('Build Docker Image') {
            steps {
                bat 'docker version'
                bat "docker build -t %IMAGE%:%TAG% ."
                bat 'docker images'
            }
        }
        
        stage('Test Application') {
            steps {
                bat """
                docker rm -f test_container 2>nul || echo "Nettoyage..."
                docker run -d --name test_container -p 8081:80 %IMAGE%:%TAG%
                timeout /t 3 /nobreak
                curl http://localhost:8081 > test_output.html 2>nul
                if exist test_output.html (
                    echo "✅ Application fonctionne!"
                    docker rm -f test_container
                ) else (
                    echo "❌ Erreur: application non accessible"
                    exit 1
                )
                """
            }
        }
        
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    bat """
                    echo %DOCKER_PASS% | docker login -u %DOCKER_USER% --password-stdin
                    docker tag %IMAGE%:%TAG% %IMAGE%:latest
                    docker push %IMAGE%:%TAG%
                    docker push %IMAGE%:latest
                    docker logout
                    echo "✅ Images poussées sur Docker Hub!"
                    """
                }
            }
        }
    }
    
    post {
        always {
            bat 'docker rm -f test_container 2>nul || echo "Nettoyage terminé"'
        }
        success {
            echo '🎉 TP3 Réussi! Pipeline terminé avec succès.'
        }
        failure {
            echo '❌ Le pipeline a échoué. Vérifiez les logs.'
        }
    }
}
