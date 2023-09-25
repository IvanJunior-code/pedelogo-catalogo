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

    }
}