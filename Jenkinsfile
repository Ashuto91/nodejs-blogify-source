pipeline {
    agent any
    tools {
        nodejs 'nodejs'
    }
    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }
    stages {
        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Git checkout') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/Ashuto91/nodejs-blogify-source.git']])
            }
        }
        stage('Install Dependencies') {
            steps {
                sh 'npm install'
                sh 'npm install mongodb'
                sh 'npm install dotenv --save'
                sh 'npm install --save-dev webpack'
                sh 'npm install --save-dev webpack-cli'
                sh 'npm ci'
            }
        }
        stage('Run Tests') {
            steps {
                sh 'npm test'
            }
        }
        stage('File System Scan') {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.html ."
            }
        }
        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \ 
                          -Dsonar.projectKey=blogify \ 
                          -Dsonar.projectName=blogify \ 
                          -Dsonar.sources=. \ 
                          -Dsonar.exclusions=**/node_modules/** \ 
                          -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info'''
                }
            }
        }
        stage('Quality Gate') {
            steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonar-token'
                }
            }
        }
        stage('Build & Push Docker Image') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'docker-cred', toolName: 'docker') {
                        sh "docker build -t ashuto91/blogify:latest ."
                        sh "docker push ashuto91/blogify:latest"
                    }
                }
            }
        }
    }
}