@Library('my-shared-lib@main') _ 

pipeline {
    agent { label 'agent1' }
    
    environment {
        IMAGE_NAME = "alia153/jenkins-ivolve"
        DEPLOYMENT_FILE="deploymn.yaml"
    }
    
    tools {
        maven 'mvn'  
    }
    
    stages {

        stage('Clone Repository') {
            steps {
                echo "Building branch: ${env.BRANCH_NAME}"
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
        }



        stage('Deploy to Kubernetes') {
            steps {
                dir('Jenkins_App') {
                    deployToK8s( DEPLOYMENT_FILE, "kubeconfig", env.BRANCH_NAME)
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
