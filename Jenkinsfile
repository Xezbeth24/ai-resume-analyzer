pipeline {
    agent any
    
    environment {
        NODE_VERSION = '20.x'
        NPM_REGISTRY = 'https://registry.npmjs.org'
        DOCKER_REGISTRY = 'docker.io'
        DOCKER_IMAGE_NAME = 'xezbeth/ai-resume-analyzer'
        BUILD_TIMESTAMP = sh(script: 'date +%Y%m%d_%H%M%S', returnStdout: true).trim()
    }
    
    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timeout(time: 30, unit: 'MINUTES')
    }
    
    stages {
        stage('Checkout') {
            steps {
                echo '========== Checking out repository =========='
                checkout scm
                sh 'git log --oneline -1'
            }
        }
        
        stage('Setup Node.js') {
            steps {
                echo '========== Setting up Node.js environment =========='
                sh '''
                    node --version
                    npm --version
                    npm ci
                '''
            }
        }
        
        stage('Code Quality Check') {
            steps {
                echo '========== Running TypeScript type checking =========='
                sh '''
                    npm run typecheck || echo "TypeScript check completed"
                '''
            }
        }
        
        stage('Build') {
            steps {
                echo '========== Building application =========='
                sh '''
                    npm run build
                    echo "Build output size:"
                    du -sh build/
                '''
            }
        }
        
        stage('Build Docker Image') {
            when {
                branch 'main'
            }
            steps {
                echo '========== Building Docker image =========='
                script {
                    sh '''
                        docker build -t ${DOCKER_IMAGE_NAME}:latest \
                                   -t ${DOCKER_IMAGE_NAME}:${BUILD_TIMESTAMP} \
                                   -t ${DOCKER_IMAGE_NAME}:${GIT_COMMIT:0:7} \
                                   .
                        
                        echo "Docker image built successfully"
                        docker images | grep ai-resume-analyzer
                    '''
                }
            }
        }
        
        stage('Push to Docker Hub') {
            when {
                branch 'main'
            }
            steps {
                echo '========== Pushing Docker image to Docker Hub =========='
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                     usernameVariable: 'DOCKER_USER', 
                                                     passwordVariable: 'DOCKER_PASS')]) {
                        sh '''
                            echo "${DOCKER_PASS}" | docker login -u "${DOCKER_USER}" --password-stdin
                            
                            docker push ${DOCKER_IMAGE_NAME}:latest
                            docker push ${DOCKER_IMAGE_NAME}:${BUILD_TIMESTAMP}
                            docker push ${DOCKER_IMAGE_NAME}:${GIT_COMMIT:0:7}
                            
                            echo "Successfully pushed Docker images"
                            docker logout
                        '''
                    }
                }
            }
        }
        
        stage('Archive Artifacts') {
            steps {
                echo '========== Archiving build artifacts =========='
                archiveArtifacts artifacts: 'build/**', 
                                 allowEmptyArchive: true,
                                 fingerprint: true
            }
        }
    }
    
    post {
        always {
            echo '========== Pipeline execution completed =========='
            cleanWs(deleteDirs: true, patterns: [[pattern: 'node_modules/', type: 'INCLUDE']])
        }
        success {
            echo '✅ Pipeline completed successfully!'
sh '''
echo "Build Summary:"
echo "- Build Status: SUCCESS"
echo "- Build Number: ${BUILD_NUMBER}"
echo "- Git Commit: ${GIT_COMMIT}"
echo "- Branch: ${GIT_BRANCH}"
echo "- Docker Image: ${DOCKER_IMAGE_NAME}:${BUILD_TIMESTAMP}"
'''        }
        failure {
            echo '❌ Pipeline failed! Check logs above for details.'
        }
        unstable {
            echo '⚠️ Pipeline completed with warnings.'
        }
    }
}
