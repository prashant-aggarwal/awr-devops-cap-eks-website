pipeline {
    agent any

	// Set the environment variables
    environment {
        PATH = "${env.HOME}/bin:${env.PATH}"
		env.IMAGE_REPO = "${env.IMAGE_REGISTRY}/${env.IMAGE_NAME}"
	}

	// Multistage pipeline
    stages {
		// Stage 0 - Setup variables
		stage('Setup variables') {
            steps {
                script {
                    echo "Using config:"
                    echo "  AWS_REGION:   ${env.AWS_REGION}"
                    echo "  CLUSTER_NAME: ${env.CLUSTER_NAME}"
                    echo "  ROLE_ARN:     ${env.ROLE_ARN}"
                    echo "  IMAGE_NAME:   ${env.IMAGE_NAME}"
                    echo "  IMAGE_TAG:    ${env.IMAGE_TAG}"
                    echo "  IMAGE_REGISTRY: ${env.IMAGE_REGISTRY}"
                    echo "  IMAGE_REPO:   ${env.IMAGE_REPO}"
                    echo "  WEB_DEPLOY:   ${env.WEB_DEPLOY}"
                }
            }
        }

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
					if ! command -v kubectl >/dev/null 2>&1; then
						echo "Installing kubectl..."
						curl -O https://s3.us-west-2.amazonaws.com/amazon-eks/1.31.2/2024-11-15/bin/linux/amd64/kubectl
						chmod +x ./kubectl
						mkdir -p $HOME/bin
						cp ./kubectl $HOME/bin/kubectl
					else
						echo "kubectl is already installed: $(kubectl version --client)"
					fi
                '''
            }
        }

		// Stage 3 - Build and Push Docker Image to ECR
		stage('Build & Push to ECR') {
			steps {
				script {
					withAWS(region: "${env.AWS_REGION}", credentials: 'AWS') {
						sh '''
							echo "Logging into Amazon ECR..."
							aws ecr get-login-password --region ${AWS_REGION} | \
							docker login --username AWS --password-stdin ${IMAGE_REGISTRY}

							echo "Building Docker image..."
							docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .

							echo "Tagging image for ECR..."
							docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_REPO}:${IMAGE_TAG}

							echo "Pushing image to ECR..."
							docker push ${IMAGE_REPO}:${IMAGE_TAG}
						'''
					}
				}
			}
		}

		// Stage 4 - Deploy web application on EKS Cluster
        stage('Deploy Web Application') {
            steps {
				script {
				// Install AWS Steps plugin to make this work
				withAWS(region: "${env.AWS_REGION}", credentials: 'AWS') {
						try {
							sh '''
								cd deploy
								# Use envsubst to replace placeholders
								sed "s|\\${IMAGE_NAME}|${IMAGE_REPO}|g" ${WEB_DEPLOY}.yaml | \
  								sed "s|\\${IMAGE_TAG}|${IMAGE_TAG}|g" > ${WEB_DEPLOY}-rendered.yaml
								aws eks update-kubeconfig --name ${CLUSTER_NAME} --region ${AWS_REGION} --role-arn ${ROLE_ARN}
								kubectl apply -f web-service.yaml
								kubectl apply -f ${WEB_DEPLOY}-rendered.yaml
								# kubectl apply -f web-deployment.yaml
								# kubectl delete -f web-deployment-v2.yaml
								
								echo "Waiting for EXTERNAL-IP of 'events-web-svc'..."

								for i in $(seq 1 60); do
									HOSTNAME=$(kubectl get svc events-web-svc -o jsonpath="{.status.loadBalancer.ingress[0].hostname}" 2>/dev/null)
									if [ -n "$HOSTNAME" ]; then
										EXTERNAL_IP="$HOSTNAME"
										echo "EXTERNAL-IP is available (hostname): $EXTERNAL_IP"
										break
									fi
									echo "Still waiting... ($i/60)"
									sleep 10
								done

								if [ -z "$EXTERNAL_IP" ]; then
									echo "Timed out waiting for EXTERNAL-IP. Exiting with error."
									exit 1
								fi


								kubectl get svc
								kubectl get pods
							'''
						} catch (exception) {
							error("Deployment failed: ${e}")
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
		success {
            echo 'Pipeline completed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}