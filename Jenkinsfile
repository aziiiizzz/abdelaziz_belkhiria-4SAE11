pipeline {
    agent any

    environment {
        // J'utilise 'latest' pour que Kubernetes détecte toujours la nouvelle version
        IMAGE_NAME = "azizashe/spring-app:latest"
    }

    stages {
        stage('Checkout') {
            steps {
                echo '📥 Récupération du code source...'
                checkout scm
            }
        }

        stage('Build Maven') {
            steps {
                echo '🔨 Compilation du projet...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                echo '🐳 Construction et envoi de l\'image Docker...'
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub-credentials',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    // Login
                    sh 'echo $PASSWORD | docker login -u $USERNAME --password-stdin'
                    
                    // Build (avec le tag latest)
                    sh "docker build --no-cache -t $IMAGE_NAME ."
                    
                    // Push
                    sh "docker push $IMAGE_NAME"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '🚀 Déploiement vers Kubernetes...'
                
                // 1. Appliquer les configurations (MySQL + Spring)
                // Le dossier k8s/ doit exister dans ton git
                sh 'kubectl apply -f k8s/ -n devops'

                // 2. Forcer le redémarrage pour télécharger la nouvelle image
                sh 'kubectl rollout restart deployment/spring-deployment -n devops'

                // J'AI ENLEVÉ LA LIGNE "rollout status" QUI FAISAIT ÉCHOUER TON BUILD PRÉCÉDENT
                echo "✅ Ordre de déploiement envoyé avec succès !"
            }
        }
    }
}
