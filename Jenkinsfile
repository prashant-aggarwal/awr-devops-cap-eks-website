pipeline {
    agent any

	// Set the environment variables
    environment {
        PATH = "${env.HOME}/bin:${env.PATH}"
    }

	// Multistage pipeline
    stages {
		// Stage 1 - Checkout code repository
        stage('Checkout Code') {
            steps {
                git branch: 'main',
                    credentialsId: 'Github',
                    url: 'https://github.com/prashant-aggarwal/awr-devops-cap-eks-website.git'
            }
        }
		
		// Stage 2 - Install AWS CLI
        stage('Install AWS CLI') {
			steps {
				sh '''
					curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
					unzip awscliv2.zip
					./aws/install
					export PATH=$HOME/bin:$PATH
					aws --version
				'''
			}
		}

		// Stage 2 - Install kubectl
        stage('Install kubectl') {
            steps {
                sh '''
					echo "Installing kubectl..."
					curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.31.2/2024-11-15/bin/linux/amd64/kubectl
					chmod +x ./kubectl
					mkdir -p $HOME/bin
					cp ./kubectl $HOME/bin/kubectl
					export PATH=$HOME/bin:$PATH
					kubectl version --client
                '''
            }
        }

		// Stage 3 - Deploy web application on EKS Cluster
        stage('Destroy Web Application') {
            steps {
				script {
				// Install AWS Steps plugin to make this work
				withAWS(region: "${AWS_REGION}", credentials: 'AWS') {
						try {
							sh '''
								cd app
								aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${AWS_REGION} --${ROLE_ARN}
								kubectl apply -f web-service.yaml
								kubectl apply -f web-deployment.yaml
								kubectl get svc
							'''
						} catch (exception) {
							echo "❌ Failed to deploy web application on EKS cluster: ${exception}"
							error("Halting pipeline due to web application deployment failure.")
						}
					}
                }
            }
        }
    }

    // Cleanup the workspace in the end
	post {
        always {
            cleanWs()
        }
    }
}
