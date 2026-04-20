pipeline {
    agent any
    environment {
        IMAGE_NAME = "nestjs-app-ds419547"
        BUILDER_IMAGE = "nestjs-builder"
        CONTAINER_NAME = "nestjs-instance-7"
    }
    stages {
        stage('Clean Workspace') {
            steps {
                echo 'Cleaning workspace...'
                deleteDir()
            }
        }
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build Builder Image') {
            steps {
                echo 'Creating Builder Image (BLDR)...'
                // Budujemy tylko pierwszy etap Dockerfile (target: build)
                sh "docker build --target build -t ${BUILDER_IMAGE}:${BUILD_NUMBER} ."
            }
        }
        stage('Test') {
            steps {
                echo 'Running tests in builder image...'
                sh "docker run --rm ${BUILDER_IMAGE}:${BUILD_NUMBER} npm test"
            }
        }
        stage('Build Runtime Image') {
            steps {
                echo 'Creating final Runtime Image...'
                sh "docker build -t ${IMAGE_NAME}:${BUILD_NUMBER} ."
            }
        }
        stage('Deploy (Sandbox)') {
            steps {
                echo 'Deploying to sandbox container...'
                // Czyścimy oba możliwe kontenery (stary z lab 6 i nowy z lab 7)
                sh "docker stop nestjs-instance || true"
                sh "docker rm nestjs-instance || true"
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"
                
                sh "docker run -d --name ${CONTAINER_NAME} -p 3005:3000 ${IMAGE_NAME}:${BUILD_NUMBER}"
            }
        }
        stage('Verification (Smoke Test)') {
            steps {
                echo 'Verifying deployment...'
                sleep 5
                sh "curl -f http://localhost:3005 || (docker logs ${CONTAINER_NAME} && exit 1)"
            }
        }
    }
    post {
        always {
            echo 'Archiving logs and cleaning up...'
            sh "docker logs ${CONTAINER_NAME} > app-logs-${BUILD_NUMBER}.log"
            archiveArtifacts artifacts: "*.log", fingerprint: true
            // sh "docker stop ${CONTAINER_NAME} || true"
        }
        success {
            echo 'Pipeline finished successfully!'
        }
    }
}
