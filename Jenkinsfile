pipeline {
    agent any

    stages {
        stage('Checkout Source') {
            steps {
                git url: 'https://github.com/tscampos/Jenkins.git', branch: 'main'
            }
        }

        stage('Build Images') {
            steps {
                script {
                    dockerapp = docker.build("tscampos/api-produto:${env.BUILD_ID}",
                      '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
                }
            }
        }

        stage('Push Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
                    dockerapp.push('latest')
                    dockerapp.push("${env.BUILD_ID}")
                    }
                }
            }
        }

        stage('Deploy Kubernetes') {
            agent {
                kubernetes {
                    cloud 'kubernetes'
                }
            }
            environment {
                tag_version = "${env.BUILD_ID}"
            }
            steps {
                script {
                    sh 'sed -i "s/{{tag}}/$tag_version/g ./k8s/api/deployment.yml'
                    sh 'cat ./k8s/api/deployment.yml'
                    kubernetesDeploy(configs: '**/k8s/**', kubeconfigId: 'config')
                }    
            }
        }
    }
}