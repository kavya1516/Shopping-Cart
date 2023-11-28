 pipeline {
    agent any

    environment {
        registryName = "shopimageacrjenkins"
        registryCredential = "ACR"
        registryUrl = 'shopimageacrjenkins.azurecr.io'
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout code from your Git repository
                git branch: 'main', changelog: false, credentialsId: 'bafe1474-147d-4685-ad51-966ef5606ec4', poll: false, url: 'https://github.com/kavya1516/Shopping-Cart.git'
            }
        }

        stage('Compile') {
            steps {
                bat "mvn clean compile -DskipTests=true"
            }
        }

        stage('OWASP Scan') {
            steps {
                // Perform OWASP Dependency Check
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'DependencyCheck'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('BUILD') {
            steps {
                bat "mvn clean package -DskipTests=true"
            }
        }

        stage('Build Docker image') {
            steps {
                script {
                    dockerImage = docker.build("shopping-cart:latest", "-f docker/Dockerfile .")
                }
            }
        }

        stage('Push Image to ACR') {
            steps {
                script {
                    docker.withRegistry("http://${registryUrl}", registryCredential) {
                        dockerImage.push()
                    }
                }
            }
        }

        stage('Deployment to Kubernetes') {
            steps {
                script {
                    kubernetesDeploy configs: 'deploymentservice.yml',  kubeconfigId: 'kube'
                }
            }
        }
    }
}
