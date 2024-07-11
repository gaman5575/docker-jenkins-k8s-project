pipeline {
    agent any
    parameters {
        string(name: "Docker_tag", description: "Enter the tag for docker image", defaultValue: 'latest')
    }
    stages {
        stage('Git Checkout') {
            steps {
                // Checkout the Respository containing the Dockerfile
                git branch: 'main',
                    url: 'https://github.com/gaman5575/docker-jenkins-project.git'
            }
        }
        stage('Docker image build') {
            steps {
                script {
                    // Make sure Docker is installed and configured on Jenkins
                    def dockerImage = docker.build("docker.io/gaman5575/todo-app:${params.Docker_tag}", "-f Dockerfile .")
                    docker.withRegistry('', 'docker_credentials') {
                        dockerImage.push("${params.Docker_tag}")
                    }
                }
            }
        }
        stage('Docker Image Scan For Vulnerabilities') {
            steps {
                sh "docker run -v /var/run/docker.sock:/var/run/docker.sock -v $HOME/Library/Caches:/root/.cache/ aquasec/trivy:0.51.1 image docker.io/gaman5575/todo-app:${params.Docker_tag}"
            }
        }
        stage('Update YAML Tag') {
            steps {
                script{
                    sh "sed -i 's|image:.*|image: docker.io/gaman5575/todo-app:${params.Docker_tag}|g' deployment.yaml"
                }
            }
        }
        stage('Deploy to Kubernetes Cluster'){
            steps{
                script{
                  // Retrieve Config file from jenkins credentials
                withKubeConfig([credentialsId: 'k8s-config', serverUrl: ' https://10.0.0.100:6443' ]) {
                    // Authenticate with kubernetes cluster
                    sh 'kubectl apply -f deployment.yaml'
                    }
                }
            }
        }
    }
}
