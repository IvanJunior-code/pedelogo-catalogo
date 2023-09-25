pipeline {
    agent any

    stages {
        stage('Checkout Source') {
            steps {
                git url:'https://github.com/IvanJunior-code/pedelogo-catalogo', branch:'main'
            }
        }

        stage('Build image') {
            steps {
                script {
                    dockerapp = docker.build("ivanjuniordocker/api-produto:${env.BUILD_ID}",
                      '-f ./src/PedeLogo.Catalogo.Api/Dockerfile .')
                }
            }
        }

        stage('Push image') {
            steps {
                script {
                    docker.withRegistry('https://registry.docker.com', 'dockerhub') {
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
                    sh 'sed -i "s/{{tag}}/$tag_version/g" ./k8s/api/deployment.yaml'
                    sh 'cat ./k8s/api/deployment.yaml'
                    //kubernetesDeploy(configs: '**/k8s/**', kubeconfigId: 'kubeconfig')
                    kubernetes.Deploy(configs: '**/k8s/**', kubeconfigId: 'kubeconfig')
                }
            }
        }

    }
}