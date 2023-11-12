pipeline {
    agent any
    
    tools {
        jdk 'jdk11'
        maven 'maven3'
    }
    
    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'master', url: 'https://github.com/prakashbelote/shopping-cart.git'
            }
        }
        
        stage('Compile') {
            steps {
                sh "mvn clean compile"
            }
        }
        stage('Build') {
            steps {
                sh "mvn clean package -DskipTests=true"
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '685ed3c8-3fd1-48fa-9ca4-a7e88d5e3e82', toolName: 'docker') {
                        sh "docker build -t shopping-cart -f docker/Dockerfile ."
                        sh "docker tag shopping-cart prakashbelote/shopping-cart:latest"
                    }
                }
            }
        }
        
        stage('Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '685ed3c8-3fd1-48fa-9ca4-a7e88d5e3e82', toolName: 'docker') {
                        sh "docker push prakashbelote/shopping-cart:latest"
                    }
                }
            }
        }
        
        stage('Deploy To Docker Container') {
            steps {
                script {
                    withDockerRegistry(credentialsId: '685ed3c8-3fd1-48fa-9ca4-a7e88d5e3e82', toolName: 'docker') {
                        sh "docker run -d --name shopping -p 8070:8070 prakashbelote/shopping-cart:latest"
                    }
                }
            }
        }
    }
}
