@Library('my-shared-lib') _ 

pipeline {
    agent { label 'agent1' }
    
    environment {
        IMAGE_NAME = "alia153/jenkins-ivolve"
        
    }
    
    tools {
        maven 'mvn'  
    }
    
    stages {

        stage('Clone Repository') {
            steps {
                git url: 'https://github.com/Aliabdrabo/iVolve-DevOps.git', branch: 'main'
            }
        }

        stage('Run Unit Tests') {
            steps {
                dir('Jenkins_App') {
                    unitTests()
                }
            }
        }

        stage('Build App') {
            steps {
                dir('Jenkins_App') {
                    buildApp()
                }
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                dir('Jenkins_App') {
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                    buildAndPushImage("${IMAGE_NAME}", "${BUILD_NUMBER}")
                }
            }
        }

        stage('Update Deployment') {
            steps {
                dir('Jenkins_App') {
                    sh "sed -i 's|image:.*|image: ${IMAGE_NAME}:v${BUILD_NUMBER}|' deployment.yaml"
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                dir('Jenkins_App') {
                    deployToK8s('deployment.yaml', "kubeconfig")
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

