pipeline {
    agent any
    environment {
        IMAGE_NAME = "azizashe/spring-app:latest"
        DOCKER_CRED_ID = "docker-hub-credentials" 
        SONAR_TOKEN = "sqa_bc27afe4e67bf8821de0d95d5e9631271f0e12cf"
    }
    stages {
        stage('Checkout Code') {
            steps {
                echo '📥 Récupération du code source...'
                git branch: 'main', url: 'https://github.com/aziiiizzz/abdelaziz_belkhiria-4SAE11.git'
            }
        }
        stage('Build Maven') {
            steps {
                echo '🔨 Compilation du JAR...'
                sh 'mvn clean package -DskipTests'
            }
        }
        stage('Build & Push Docker') {
            steps {
                echo '🐳 Construction et Push de l\'image Docker (Via Shell)...'
                withCredentials([usernamePassword(credentialsId: DOCKER_CRED_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker build --no-cache -t $IMAGE_NAME ."
                    sh "docker push $IMAGE_NAME"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo '🚀 Déploiement sur K8s (Namespace: devops)...'
                sh 'kubectl create namespace devops --dry-run=client -o yaml | kubectl apply -f -'
                sh 'kubectl apply -f k8s/ -n devops'
                sh 'kubectl rollout restart deployment/spring-deployment -n devops' 
                echo "✅ Ordre de déploiement envoyé avec succès !"
            }
        }
    } 

    post {
        success {
            echo "✅ Pipeline RÉUSSI ! Application déployée."
        }
        failure {
            echo "❌ ÉCHEC du pipeline. Vérifie les logs."
        }
    }
}
