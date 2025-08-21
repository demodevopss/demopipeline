pipeline {
    agent {
        label 'my-agent'
    }
    environment {
        APP_NAME = "demopipeline"
        RELEASE = "1.0"
        DOCKER_USER = "devopsserdar"
        DOCKER_LOGIN = "dockerhub"
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}.${BUILD_NUMBER}"
    }
    tools {
        jdk 'JDK21'
        maven 'Maven3'
    }
    stages {
        stage('Cleanup Workspace') {
            steps {
                cleanWs()
            }
        }
        stage('Checkout from SCM') {
            steps {
                git branch: 'main', credentialsId: 'github', url: 'https://github.com/demodevopss/demopipeline'
            }
        }
        stage('Build Maven') {
            steps {
                sh 'mvn clean package'
            }
        }
        stage('Test Application') {
            steps {
                sh 'mvn test'
            }
        }
        /*
        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                        sh """
                            mvn sonar:sonar \
                            -Dsonar.projectKey=${APP_NAME} \
                            -Dsonar.projectName=${APP_NAME} \
                            -Dsonar.projectVersion=${IMAGE_TAG}
                        """
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            echo "Pipeline unstable due to quality gate failure: ${qg.status}"
                            // abortPipeline: false means pipeline continues even if quality gate fails
                        }
                    }
                }
            }
        }
        */
        stage('Build & Push Docker Image to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_LOGIN) {
                        def docker_image = docker.build "${IMAGE_NAME}"
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }
        stage("Trivy Scan") {
            steps {
                script {
                    sh ("docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME}:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table")
                }
            }
        }
        stage('Cleanup Artifacts') {
            steps {
                script {
                    // For Unix (Mac/Linux)
                    if (isUnix()) {
                        sh label: 'Docker cleanup on Unix', script: '''
                            echo "Cleaning up Docker artifacts on Unix..."
                            docker rmi ${IMAGE_NAME}:${IMAGE_TAG} || true
                            docker rmi ${IMAGE_NAME}:latest || true
                            # Clean any remaining images with our app name
                            REMAINING_IMAGES=$(docker images --format '{{.Repository}}:{{.Tag}}' | grep 'demopipeline' || true)
                            if [ ! -z "$REMAINING_IMAGES" ]; then
                                echo "Removing remaining images: $REMAINING_IMAGES"
                                docker rmi $REMAINING_IMAGES || true
                            fi
                            # Clean containers and volumes
                            docker container rm -f $(docker container ls -aq) || true
                            docker volume prune -f || true
                        '''
                    }
                    // For Windows
                    else {
                        bat label: 'Docker cleanup on Windows', script: '''
                            echo Cleaning up Docker artifacts on Windows...
                            docker rmi %IMAGE_NAME%:%IMAGE_TAG% || exit 0
                            docker rmi %IMAGE_NAME%:latest || exit 0
                            REM Clean any remaining images with our app name
                            for /f "tokens=*" %%i in ('docker images --format "{{.Repository}}:{{.Tag}}" ^| findstr "demopipeline" 2^>nul') do docker rmi %%i || exit 0
                            REM Clean containers and volumes
                            docker container rm -f %%(docker container ls -aq) || exit 0
                            docker volume prune -f || exit 0
                        '''
                    }
                }
            }
        }
    }
}
