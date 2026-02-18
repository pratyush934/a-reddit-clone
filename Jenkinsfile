pipeline {
    agent any

    tools {
        jdk 'jdk'
        nodejs 'nodejs'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        APP_NAME     = "reddit-clone-pipeline"
        RELEASE      = "1.0.0"
        DOCKER_USER  = "pratyush934"
        IMAGE_NAME   = "${DOCKER_USER}/${APP_NAME}"
        IMAGE_TAG    = "${RELEASE}-${BUILD_NUMBER}"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/pratyush934/a-reddit-clone.git'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('sonar') {
                    sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectName=${APP_NAME} \
                        -Dsonar.projectKey=${APP_NAME}
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                waitForQualityGate abortPipeline: false
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm install'
            }
        }

        stage('Trivy FS Scan') {
            steps {
                sh 'trivy fs . --severity HIGH,CRITICAL --format table > trivyfs.txt'
            }
        }

        stage('Build & Push Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', 'docker-cred') {
                        def appImage = docker.build("${IMAGE_NAME}:${IMAGE_TAG}")
                        appImage.push()
                        appImage.push('latest')
                    }
                }
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh """
                trivy image ${IMAGE_NAME}:latest \
                --severity HIGH,CRITICAL \
                --no-progress \
                --format table > trivyimage.txt
                """
            }
        }

        stage('Cleanup Docker Images') {
            steps {
                sh """
                docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true
                docker rmi ${IMAGE_NAME}:latest || true
                """
            }
        }
    }
         post {
            always {
               emailext attachLog: true,
                   subject: "'${currentBuild.result}'",
                   body: "Project: ${env.JOB_NAME}<br/>" +
                       "Build Number: ${env.BUILD_NUMBER}<br/>" +
                       "URL: ${env.BUILD_URL}<br/>",
                   to: '2022594965.pratyush@ug.sharda.ac.in',                              
                   attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
            }
     }
}
