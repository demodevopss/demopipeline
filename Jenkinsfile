pipeline {
    agent any

    tools{
        jdk 'JDK21'
        maven 'Maven3'
    }

    stages {


        stage('Build Maven') {
            steps {
                checkout scmGit(branches: [[name: '*/main']], extensions: [], userRemoteConfigs: [[url: 'https://github.com/demodevopss/demopipeline']])

                sh 'mvn clean install'
            }
        }


        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }


        stage('Docker Image') {
           steps {
               //  sh 'docker build  -t demodevopss/demopipeline:latest  .'
                  bat 'docker build  -t demodevopss/demopipeline:latest  .'
           }
        }


        stage('Docker Image to DockerHub') {
            steps {
                script{
                    withCredentials([string(credentialsId: 'dockerhub', variable: 'dockerhub')]) {

                        //  sh 'echo docker login -u devopsserdar -p DOCKERHUB_TOKEN'
                         sh 'echo docker login -u devopsserdar -p ${dockerhub}'
                        // sh 'docker image push  devopsserdar/my-application:latest'
                    }
                }
            }
        }


        stage('Deploy to Kubernetes'){
            steps{
                kubernetesDeploy (configs: 'deployment-service.yml', kubeconfigId: 'kubernetes')
            }
        }


       stage('Docker Image to Clean') {
           steps {
               sh 'docker rmi devopsserdar/my-application:latest'

               sh 'docker image prune -f'
           }
       }


    }
}
