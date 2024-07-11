pipeline {
    agent any
    parameters {
        string(name: "Docker_tag", description: "Enter the tag for docker image", defaultValue: 'latest')
    }
    stages {
        stage('Git Checkout') {
            steps {
                // Checkout the Repository containing the Dockerfile
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
                script {
                    // Use bash shell instead of default shell
                    sh '''
                        #!/bin/bash
                        # Navigate to the correct directory
                        cd ~/docker-k8s-jenkins-project/docker-jenkins-k8s-project
                        
                        # Add a debugging step to list files in the current directory
                        ls -l
                        
                        # Check if deployment.yaml file exists
                        if [ -f deployment.yaml ]; then
                            sed -i 's|image:.*|image: docker.io/gaman5575/todo-app:${params.Docker_tag}|g' deployment.yaml
                        else
                            echo "Error: deployment.yaml file not found"
                            exit 1
                        fi
                    '''
                }
            }
        }
        stage('Deploy to Kubernetes Cluster') {
            steps {
                script {
                    // Retrieve Config file from jenkins credentials
                    withKubeConfig([credentialsId: 'k8s-config', serverUrl: 'https://10.0.0.100:6443']) {
                        // Authenticate with kubernetes cluster
                        sh '''
                            #!/bin/bash
                            # Navigate to the correct directory
                            cd ~/docker-k8s-jenkins-project/docker-jenkins-k8s-project
                            kubectl apply -f deployment.yaml
                        '''
                    }
                }
            }
        }
    }
}
