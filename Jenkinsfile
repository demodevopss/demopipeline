pipeline {
    agent {
        label 'Jenkins-Agent'
    }
    environment {
        APP_NAME = "demopipeline"
        RELEASE = "1.0"
        DOCKER_USER = "demodevopss"
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
/*        stage("SonarQube Analysis") {
            steps {
                script {
                    withSonarQubeEnv(credentialsId: 'sonarqube-token') {
                        sh "mvn sonar:sonar"
                    }
                }
            }
        }
        stage("Quality Gate") {
            steps {
                script {
                    timeout(time: 10, unit: 'MINUTES') { // 10 dakika zaman aşımı
                        waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube-token'
                    }
                }
            }
        }
*/
        stage('Build & Push Docker Image to DockerHub') {
            steps {
                script {
                    docker.withRegistry('', DOCKER_LOGIN) {
                        docker_image = docker.build "${IMAGE_NAME}"
                        docker_image.push("${IMAGE_TAG}")
                        docker_image.push("latest")
                    }
                }
            }
        }
        stage("Trivy Scan") {
            steps {
                script {
                    sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image demodevopss/demopipeline:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table')
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
                            docker rmi $(docker images --format '{{.Repository}}:{{.Tag}}' | grep 'devops-003-pipeline-aws' || echo "No images found") || true
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
                            docker rmi %%(docker images --format "{{.Repository}}:{{.Tag}}" ^| findstr "devops-003-pipeline-aws" || echo "No images found") || exit 0
                            docker container rm -f %%(docker container ls -aq) || exit 0
                            docker volume prune -f || exit 0
                        '''
                    }
                }
            }
        }
    }
}
/*
        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                    sh "docker rmi $(docker images --format '{{.Repository}}:{{.Tag}}' | grep 'devops-003-pipeline-aws')"
                    sh "docker container rm -f $(docker container ls -aq)"
                    sh "docker volume prune"


                    // Agent makinesi zamanla dolacak. Docker şişecek dolacak. Temizlik yapmanız lazım.
                    // Agent makinede temizlik için yeriniz azalmışsa şu komutları kulanın lütfen.
                    // Hatta mümkünse bu kodları buraya uyarlayın lütfen.
                    /*
                    docker rmi $(docker images --format '{{.Repository}}:{{.Tag}}' | grep 'devops-003-pipeline-aws')

                    docker container rm -f $(docker container ls -aq)

                    docker volume prune


                }
            }
        }


        stage('Deploy Kubernetes') {
            steps {
                script {
                    kubernetesDeploy(configs: 'deploymentservice.yaml', kubeconfigId: 'kubernetes')
                }
            }
        }
        stage('Docker Image to Clean') {
            steps {
                bat 'docker image prune -f'
            }
        }

    }
}
*/