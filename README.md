## Deploy an application in an existing EKS cluster using kubectl commands through a multi-branch pipeline
Setup the necessary configuration files for deployment of the desired application to an existing EKS cluster.

#### Create a Jenkinsfile for running kubectl commands for deployment of application in EKS cluster:
1. Create a pipeline.
2. Set relevant environment variables.
3. Create various stages within the pipeline:
   - Install AWS CLI.
   - Install kubectl.
   - Deploy the application in EKS cluster using kubectl commands. The kubectl commands will only work if the updated kubeconfig file is available which can be generated using **aws eks update-kubeconfig** command.
   - NOTE that the scripts cannot be executed using sudo or root access.
  
#### Setup Jenkins pipeline:
1. Login to the Jenkins server.
2. Install necessary plugins using Manage Jenkins -> Plugins option.
3. Add Github credentials in Jenkins -> Manage Jenkins -> Credentials -> Add Credentials -> Kind (Username and Password). Generate a personal access token in your Github account and use it as a password instead.
4. You can also setup Github Webhook in Auto Mode for Jenkins server using Github App. After successful configuration of Github App in both the Github account and Jenkins pipeline, there is no need to manually configure Webhooks in each of the individual repositories.
5. Add DockerHub credentials in Jenkins -> Manage Jenkins -> Credentials -> Add Credentials -> Kind (Username and Password).
6. Add AWS credentials in Jenkins -> Manage Jenkins -> Credentials -> Add Credentials -> Kind (AWS Credentials).
7. Create a Jenkins pipeline using **New Item** option.
8. Set this repository as the SCM source with necessary GIT settings.
9. Set the Script Path as Jenkinsfile using Build Configuration -> Mode (by Jenkinsfile).
10. Save the changes, goto the pipeline associated with the desired branch and click **Build Now** to trigger the pipeline.
11. Check the Console Output associated with the lastest job for verification. You will find the EXTERNAL-IP value with a DNS name, suffix port number 80 e.g. http://a08632508ba4d4c8ab388a87bd938639-236338372.us-east-1.elb.amazonaws.com:80/ and open it in a browser to test the application.
12. Also verify the EKS cluster in the AWS management console in the selected region.

#### Autoscaling and readiness/liveliness checks:
1. Update the configuration files for changing the application behavior.
2. Commit the changes and wait for the build trigger.
3. Check the Console Output associated with the lastest job for verification.
4. Verify the Pods in EKS cluster in the AWS management console in the selected region.

#### Blue/green deploments:
1. Update the Jenkinsfile for deployment of another version of the application.
2. Commit the changes and wait for the build trigger.
3. Check the Console Output associated with the lastest job for verification.
4. Verify the Pods in EKS cluster in the AWS management console in the selected region.

#### Setup Kubernetes agent:
1. Setup Kubernetes agent as per the instructions in https://plugins.jenkins.io/kubernetes/.
2. The Kubernetes Cloud agent requires private key for a successful connection to the EKS cluster. Create a service account for generation of a private key and save it as a Secret Text in Jenkins to associate it with the agent.
3. You also need to add the IAM user/role to the EKS cluster using Access tab with the necessary policies attached to it. 
4. Ensure that the necessary IAM role is created with necessary permissions which can also assume role for performing operations on EKS cluster.
5. Use websockets for connection, and test the connection to ensure that the Kubernetes agent is able to communicate with the desired EKS cluster.
6. Associate the Kubernetes Cloud agent with the pipeline and update the Jenkinsfile to use kubernetes agent for deployment.
7. Commit the changes and wait for the build trigger.
8. Check the Console Output associated with the lastest job for verification.
9. Verify the Pods in EKS cluster in the AWS management console in the selected region.






