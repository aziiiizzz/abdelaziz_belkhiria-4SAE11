pipeline {
    agent any

    environment {
        IMAGE_NAME = "azizashe/spring-app:1.0.0"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Récupération du code source...'
                checkout scm
            }
        }

        stage('Build') {
            steps {
                echo 'Compilation du projet...'
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Docker Build & Push') {
            steps {
                echo 'Construction et envoi de l\'image Docker...'
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVariable: 'USERNAME',
                    passwordVariable: 'PASSWORD'
                )]) {
                    sh "docker build -t $IMAGE_NAME ."
                    sh "echo \$PASSWORD | docker login -u \$USERNAME --password-stdin"
                    sh "docker push $IMAGE_NAME"
                }
            }
        }
    }
}
