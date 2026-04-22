pipeline {
    agent any

    options {
        disableConcurrentBuilds()
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
    }

    parameters {
        choice(name: 'ENV', choices: ['dev', 'uat', 'prod'], description: 'Select Environment')
        string(name: 'BRANCH', defaultValue: 'main', description: 'Git Branch')
    }

    environment {
        APP_NAME = "my-app"
        DOCKER_IMAGE = "myrepo/my-app"
        AWS_REGION = "ap-south-1"
    }

    stages {

        stage('Checkout Code') {

        }

        stage('Build') {
            steps {
                sh '''
                    echo "Building application..."
                    mvn clean package -DskipTests
                '''
            }
        }

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('Code Quality - SonarQube') {
            steps {
                withSonarQubeEnv('sonarqube-server') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Docker Build & Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-creds') {
                        def image = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                        image.push()
                        image.push('latest')
                    }
                }
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

        stage('Deploy to Kubernetes (EKS)') {
            steps {
                withCredentials([file(credentialsId: 'kubeconfig', variable: 'KUBECONFIG')]) {
                    sh '''
                        echo "Deploying to ${ENV} environment"
                        kubectl set image deployment/${APP_NAME} ${APP_NAME}=${DOCKER_IMAGE}:${BUILD_NUMBER} --record
                    '''
                }
            }
        }

        stage('Post Deployment Verification') {
            steps {
                sh '''
                    echo "Checking rollout status..."
                    kubectl rollout status deployment/${APP_NAME}
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
()