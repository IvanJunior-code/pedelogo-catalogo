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
                    sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"'
                    sh 'curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"'
                    sh 'echo "$(cat kubectl.sha256)  kubectl" | sha256sum --check'
                    sh 'sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl'
                    sh 'chmod u+x ./kubectl'
                    withKubeConfig([credentialsId: 'kube']) {
                        sh 'kubectl apply -f ./k8s/ -R'
                    }
                }
            }
        }

    }
}