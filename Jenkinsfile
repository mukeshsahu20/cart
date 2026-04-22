pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '5'))

    }

    parameters {
        choice(name: 'ENV', choices: ['dev', 'uat', 'prod'], description: 'Select Environment')
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git Branch')
    }

    environment {
        APP_NAME = "my-app"
        DOCKER_IMAGE = "myrepo/my-app"
    }

    stages {

        stage('Checkout Code') {
            steps {
                git branch: "${params.BRANCH}",
                    url: 'https://github.com/git/git.git'   // public repo for demo
            }
        }

        stage('Build (Demo)') {
            steps {
                bat '''
                echo Building application on Windows...
                echo Simulating build process...
                '''
            }
        }

        stage('Unit Test (Demo)') {
            steps {
                bat '''
                echo Running unit tests...
                echo All tests passed!
                '''
            }
        }

        stage('Code Quality (Demo)') {
            steps {
                bat '''
                echo Running code quality checks...
                '''
            }
        }

        stage('Docker Build (Optional)') {
            when {
                expression { return false } // disable for demo if Docker not installed
            }
            steps {
                bat '''
                docker build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .
                '''
            }
        }

        stage('Approval for Production') {
            when {
                expression { params.ENV == 'prod' }
            }
            steps {
                input message: "Approve deployment to PROD?", ok: "Deploy"
            }
        }

        stage('Deploy (Demo)') {
            steps {
                bat '''
                echo Deploying %APP_NAME% to %ENV% environment...
                echo Deployment successful!
                '''
            }
        }

        stage('Post Deployment Verification') {
            steps {
                bat '''
                echo Verifying deployment...
                echo Application is running fine!
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline executed successfully!"
        }
        failure {
            echo "❌ Pipeline failed!"
        }
        always {
            cleanWs()
        }
    }
}