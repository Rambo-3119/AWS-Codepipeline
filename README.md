**AWS 4-Stage CI/CD Pipeline Setup**

This README provides step-by-step instructions to set up a 4-stage CI/CD pipeline in AWS using CodeCommit, CodeBuild, CodeDeploy, and CodePipeline. The stages include Source, Build, Test, and Deploy.


**Prerequisites**

- AWS account with access to IAM, CodeCommit, CodeBuild, CodeDeploy, CodePipeline, and EC2 services.
- Configured AWS CLI and SSH keys.

**Steps**

**Step 1: Push Code to AWS CodeCommit Repository**

1. **Create a CodeCommit Repository:**
   - Go to **AWS CodeCommit** and create a new repository.
   - Name the repository and note the SSH URL.

2. **Set Up SSH for CodeCommit:**
   - Generate an SSH key pair (if not already created).
   - In **IAM**, associate the SSH public key with your user profile.
   - Add the repository's SSH URL to your Git config with:
     ```bash
     git remote add origin ssh://git-codecommit.<region>.amazonaws.com/v1/repos/<repository-name>
     ```

3. **Push Code:**
   - Commit and push code to the CodeCommit repository.
     ```bash
     git add .
     git commit -m "Initial commit"
     git push origin main
     ```


**Step 2: Create a CodeBuild Stage**

1. **Create a Build Project in CodeBuild:**
   - Go to **AWS CodeBuild** and create a new build project.
   - Set the source provider to **CodeCommit** and select your repository.
   - Configure the environment (e.g., runtime environment, permissions).
   - Use a buildspec file (`buildspec.yaml`) or create commands inline for build steps.

2. **Define Build Artifacts:**
   - Under **Buildspec**, specify any artifacts to be generated.
   - Ensure permissions for the CodeBuild role to write to the S3 bucket if using S3 as an artifact destination.

**Step 3: Set Up an EC2 Instance for Application Deployment**

1. **Launch an EC2 Instance:**
   - Go to **AWS EC2** and launch an instance with your preferred settings (e.g., Amazon Linux 2).
   - Configure the security group to allow incoming SSH and HTTP/HTTPS traffic.
   - Connect to the instance and install any dependencies required for the application.

2. **Create an IAM Role for EC2 Instance Access:**
   - Go to **IAM**, create a new role with permissions for **CodeDeploy**.
   - Attach this role to the EC2 instance to allow deployment access.


**Step 4: Create a CodeDeploy Application and Deployment Group**

1. **Create a CodeDeploy Application:**
   - Go to **AWS CodeDeploy** and create an application for EC2/On-premises.
   - Name the application.

2. **Create a Deployment Group:**
   - In the application, create a **Deployment Group**.
   - Select **EC2 Instances** as the deployment type and assign the EC2 instance by tags or manually.
   - Attach the **CodeDeploy** IAM role.

**Step 5: Set Up the CodePipeline with Custom Stages**

1. **Create a Pipeline in CodePipeline:**
   - Go to **AWS CodePipeline** and create a new pipeline.
   - Choose **Custom Pipeline** to configure each stage.

2. **Add Source Stage:**
   - Choose **AWS CodeCommit** as the source.
   - Select the repository created earlier and the branch (e.g., `main`).

3. **Add Build Stage:**
   - Select **CodeBuild** as the provider and choose the build project created in Step 2.

4. **Add Deploy Stage:**
   - Choose **CodeDeploy** as the provider.
   - Select the application and deployment group created in Step 4.

**Step 6: Add a Test Stage in CodeBuild**

1. **Create a New Build Project for Testing in CodeBuild:**
   - Go to **AWS CodeBuild** and create another build project.
   - This time, specify a `test-buildspec.yaml` file to define the test steps.
   - Set up environment variables, build role, and permissions as required.

**Step 7: Edit Pipeline to Add the Test Stage**

1. **Edit the CodePipeline:**
   - Go to **AWS CodePipeline** and edit the pipeline.
   - Add a new **Test Stage** between Build and Deploy.

2. **Configure the Test Stage:**
   - Choose **CodeBuild** as the action provider.
   - Select the test build project created in Step 6.
   - In **Build Artifacts**, specify `test-buildspec.yaml`.

3. **Save and Execute Pipeline:**
   - Save changes and manually execute the pipeline to test the full workflow.

**Execution and Verification**

After execution, the pipeline will proceed through the following stages:

1. **Source:** Pulls the latest code from CodeCommit.
2. **Build:** Compiles the code and creates build artifacts.
3. **Test:** Runs tests as specified in the `test-buildspec.yaml`.
4. **Deploy:** Deploys the application to the EC2 instance using CodeDeploy.


**Troubleshooting**

- **Pipeline Fails:** Check logs in CodeBuild, CodeDeploy, and CloudWatch for details.
- **Permissions:** Ensure all roles and permissions are correctly set up for each service.


This setup provides a scalable, automated deployment pipeline with 4 stages, enabling continuous integration and deployment on AWS.
