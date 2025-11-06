pipeline {
    agent any
    
    environment {
        IMAGE_NAME = "alia153/jenkins-ivolve"
    }
    
    tools {
        maven 'mvn'  
    }
    
    stages {
        stage('Cloning the repo') {
            steps {
                git url: 'https://github.com/Aliabdrabo/iVolve-DevOps.git', branch: 'main'
            }
        }
        
        stage('Run Unit Test') {
            steps {
                dir('Jenkins_App') {
                    sh 'mvn test'
                }
            }
        }
        
        stage('Build the app') {
            steps {
                 dir('Jenkins_App') {
                    sh 'mvn package'
                }
            }
        }
        
        stage('Build Docker image') {
            steps {
                 dir('Jenkins_App') {
                    sh "docker build -t ${IMAGE_NAME}:v${BUILD_NUMBER} ."
                }
            }
        }
        
        stage('Push Docker image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    sh "docker push ${IMAGE_NAME}:v${BUILD_NUMBER}"
                }
            }
        }
        
        stage('Delete image locally') {
            steps {
                sh "docker rmi ${IMAGE_NAME}:v${BUILD_NUMBER}"
            }
        }
        
        stage('Update Deployment') {
            steps {
                dir('Jenkins_App') {
                    sh 'sed -i "s|image:.*|image: alia153/jenkins-ivolve:v6|" deployment.yaml'
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                 dir('Jenkins_App') {
                    withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                        sh "kubectl apply -f deployment.yaml"
                    }   
                }
            }
        }
    }  
    
    post {  
        success {
            echo "✅ Pipeline succeeded!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
        always {
            echo "🔄 Pipeline finished!"
        }
    }
}
