pipeline {
    agent any

	// Set the environment variables
    environment {
        PATH = "${env.HOME}/bin:${env.PATH}"
		CLUSTER_NAME = 'pa-cap-eks-cluster-yVMH'
		AWS_REGION = 'us-east-1'
		ROLE_ARN = 'arn:aws:iam::021668988309:role/EKSServiceDeploymentRole'
    }

	// Multistage pipeline
    stages {
		// Stage 1 - Install AWS CLI
        stage('Install AWS CLI') {
			steps {
				sh '''
					if ! command -v aws >/dev/null 2>&1; then
						echo "AWS CLI not found. Installing..."
						curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
						unzip -q awscliv2.zip
						./aws/install -i $HOME/aws-cli -b $HOME/bin
					  else
						echo "AWS CLI is already installed: $(aws --version)"
					  fi
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
        stage('Deploy Web Application') {
            steps {
				script {
				// Install AWS Steps plugin to make this work
				withAWS(region: "${AWS_REGION}", credentials: 'AWS') {
						try {
							sh '''
								cd app
								aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${AWS_REGION} --role-arn ${ROLE_ARN}
								kubectl apply -f web-service.yaml
								kubectl delete -f web-deployment-v2.yaml
								kubectl get svc
								kubectl get pods
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
