# Spring Boot CI/CD Pipeline with GitHub Actions

## This guide walks you through setting up a robust Continuous Integration (CI) and Continuous Deployment (CD) pipeline for a Java Spring Boot application using GitHub Actions.

## The pipeline includes:

## The Continuous Integration Stage

### 1. Continuous Integration (CI):
### Imagine a team of people building a house. Instead of everyone working on their own part in secret and only bringing it all together at the very end (which would be chaos!), CI is like everyone frequently sharing their completed pieces. Every time someone finishes a bit of work, they add it to a central shared blueprint. This way, if someone's new piece doesn't fit with what others have done, you find out immediately, not right before the housewarming party! It's about frequently merging code changes to detect problems early.

### 2. Code Checkout: 
### This is like getting a copy of the blueprint for the house from the central storage. When you start working, you "check out" the latest version of the code (the blueprint) from the shared system (like GitHub or GitLab) to your own computer so you can make your changes.

### 3. Java Environment Setup (JDK 21): 
### To work on a Java project, you need the right tools. Think of it as setting up your workshop with the specific hammer, saw, and other tools needed to build a Java house. JDK 21 (Java Development Kit 21) is a specific version of those tools that allows you to write and run Java programs. Without it, your computer wouldn't understand the Java code.

### 4. Dependency Caching (Maven): 
### When building a house, you might need to use pre-made parts like windows or doors. Instead of ordering them fresh every single time you want to build a house, you might keep a stock of them in a local warehouse. Similarly, in software, projects often rely on other pre-written pieces of code called "dependencies." Maven is a tool that helps manage these. "Dependency caching" means storing these pre-made code pieces on your computer so you don't have to download them from the internet every time you build your project, making the process much faster.

### Linting & Formatting (Checkstyle): 
### This is like having a set of rules for how your blueprint should look. For example, "all lines must be drawn with a ruler" or "all labels must be in this specific font." Linting checks for potential errors or bad practices in your code (like spelling mistakes in the blueprint), and formatting ensures everyone writes code in a consistent style (like making sure all your architects draw lines the same way). Checkstyle is a specific tool that helps enforce these rules for Java code.

### Building & Unit Testing (Maven): 
### "Building" is the actual construction process – taking your raw materials (your code) and assembling them into a working program. "Unit testing" is like testing each individual part of the house as you build it. Does this door open and close properly? Does this pipe hold water? You test small, isolated pieces of your code to make sure they work exactly as intended before you put them all together. Maven is the tool that orchestrates this building and testing process.

### Code Scanning (SonarCloud): 
### After you've built some parts of the house, you might bring in an inspector to check for hidden flaws or potential weaknesses, not just if the parts work, but if they're built securely and efficiently. Code scanning is an automated inspection of your code to find more complex issues like security vulnerabilities, bugs, or places where the code could be more efficient. SonarCloud is a service that performs these deep code inspections.

### Secret Scanning (Gitleaks): 
### This is like having a super-secret safe in your house blueprint, and you accidentally drew the combination on the outside! "Secrets" in software are sensitive pieces of information like passwords, API keys, or security credentials. Secret scanning is a specialized check to make sure you haven't accidentally left any of these sensitive secrets exposed in your code where others could find them. Gitleaks is a tool specifically designed for this.

## The Continuous Deployment Stage

### 1. Continuous Deployment (CD): 
### While Continuous Integration (CI) is about frequently merging code, Continuous Deployment (CD) takes it a step further. It's like having a fully automated pipeline where, once your house blueprint (code) has passed all the initial checks and tests, it's automatically built, approved, and delivered to the construction site (the live environment) without any manual intervention. The goal is to get new, working features to users as quickly and safely as possible.

### 2. Manual Approval Gate (GitHub Environments): 
### Even with full automation, sometimes you want a human to give the final "go-ahead" before something goes live, especially for critical changes or to a production environment. A "manual approval gate" is like an actual gate on the construction site that only opens when a manager gives the explicit permission. GitHub Environments is a feature within GitHub that allows you to set up these gates, so a designated person has to click "approve" before the deployment proceeds.

### 3. Docker Image Build: 
### Imagine instead of building your house piece by piece on the construction site every time, you build it entirely in a factory, package it up neatly in a container, and then ship that ready-made container to the site. A "Docker Image" is like that pre-packaged container for your software application. "Docker Image Build" is the process of taking your application code and all its necessary dependencies (libraries, tools, etc.) and bundling them into a single, self-contained, and portable unit (the Docker image). This ensures your application runs consistently, no matter where it's deployed.

### 4. Vulnerability Scanning (Trivy) on the Docker Image: 
### Before you ship that pre-built house container to the site, you'd want to scan it for any hidden defects or structural weaknesses that could make it vulnerable. "Vulnerability scanning" on a Docker image is an automated check of that pre-packaged application for known security flaws or vulnerabilities in the components it uses. Trivy is a popular open-source tool specifically designed to perform these security scans on Docker images, helping you catch potential issues before they go live.

### 5. Docker Image Push to Docker Hub: (Pulling image, stopping old container, running new one)
### Once your containerized house is built and scanned, you need a place to store it where it can be easily accessed for deployment. "Docker Hub" is like a public or private warehouse for these pre-built Docker images. "Docker Image Push" is the act of uploading your newly built and scanned Docker image to Docker Hub (or a similar container registry). From Docker Hub, it can then be easily pulled down and deployed to your servers.



## Step 1
## Project Overview & Prerequisites

### We will create a simple Spring Boot REST API that exposes a /hello endpoint. This application will then be containerized and deployed.

## Prerequisites (What you need before you start)

- Local Development Environment (Your Ubuntu VM):

- Java Development Kit (JDK) 21: Installed and configured.

- Apache Maven: Installed.

- Docker: Installed and configured (your user in the docker group).

- Git: Installed.

- Internet Access: Your VM needs a stable internet connection.

- Accounts & Services:

- GitHub Account: To host your code and run GitHub Actions.

- Docker Hub Account: To store your Docker images.

- AWS Account: With an EC2 instance (Ubuntu Server recommended) where Docker is installed.

- EC2 Security Group: Must allow inbound SSH (Port 22) from your IP and HTTP (Port 80) from 0.0.0.0/0.

- EC2 SSH Key Pair (.pem file): For connecting to your instance.

- SonarCloud Account: For static code analysis. (Free for public repositories).

## Step Two:
## Prepare Your Ubuntu VM Environment (Crucial!)

### This is the most common source of initial errors. Follow these steps meticulously to ensure your local environment is correctly set up.

### 1. Open a New Terminal Session:
If you're using SSH, close any existing sessions and reconnect. This ensures a clean shell environment.

### 2. Clean Up Any Previous Java Installations:
This helps prevent conflicts.

```
# Update package list
sudo apt update

# List any installed OpenJDK packages (look for 'openjdk-XX-jdk' or 'openjdk-XX-jre')
sudo apt list --installed | grep openjdk

# Remove any OpenJDK packages you find from the list (replace 'XX' with version numbers)
# Example: sudo apt purge openjdk-17-jdk openjdk-17-jre -y
# Example: sudo apt purge openjdk-11-jdk openjdk-11-jre -y
```

### 3. Install OpenJDK 21 (Latest LTS Version):

```
sudo apt install openjdk-21-jdk -y
```
### 4. Verify Java Installation and Find Its Path:

```
java -version
javac -version
readlink -f /usr/bin/java | sed "s:bin/java::"
```
- Verify: Both java -version and javac -version must show 21.0.x.

- Copy the path: The output of readlink -f ... will be your JAVA_HOME path (e.g., /usr/lib/jvm/java-21-openjdk-amd64/). Copy this exact path.

### 5. Configure JAVA_HOME in ~/.bashrc:
This tells Maven and other tools where to find your Java installation.

```
nano ~/.bashrc
```
- Scroll to the very end of the file.

- Delete any old JAVA_HOME or PATH exports related to Java that you might have added previously.

- Paste these lines at the end, replacing /usr/lib/jvm/java-21-openjdk-amd64 with the exact path you copied from the previous step:

```
# --- START OF JAVA_HOME CONFIGURATION ---
# Set JAVA_HOME for JDK 21
export JAVA_HOME="/usr/lib/jvm/java-21-openjdk-amd64" # <-- PASTE YOUR EXACT PATH HERE!
# Add JAVA_HOME's bin directory to your PATH, ensuring it's at the start
export PATH="$JAVA_HOME/bin:$PATH"
# --- END OF JAVA_HOME CONFIGURATION ---
```

### 6. Save and exit nano

### 7. Apply .bashrc Changes:

```
source ~/.bashrc
```
- (Or, close and re-open your terminal/SSH session for the most reliable application).

### 8. Final Java Environment Verification:

```
java -version
javac -version
echo $JAVA_HOME
echo $PATH
```
- Confirm: All outputs should clearly indicate Java 21, and your PATH should start with your Java 21 ```bin``` directory.

### 9. Install Maven:
```
sudo apt install maven -y
```

### 10. Install Docker (if not already fully working):
```
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker $(whoami) # Add your current user to the docker group
newgrp docker # Apply group membership immediately (might disconnect, reconnect if it does)
```
- After running 'newgrp docker', you may need to reconnect your SSH session. Then, verify with docker ps.


## Step Three:
## Create the Java Spring Boot Application & Dockerfile

### Now that your environment is ready, let's create the application code and its Dockerfile.

### 1.  Create Your Project Directory

```
# Go to your desired parent directory (e.g., where you keep your projects)
cd ~
mkdir Git-Action-Project
cd Git-Action-Project

# Create a new directory for this specific project
mkdir spring-ecommerce-ci-cd

# Navigate into your new project directory
cd spring-ecommerce-ci-cd

```

### 2. Create Spring Boot Project Structure and pom.xml

### We'll use Apache Maven to create the basic project structure and manage dependencies.
```
# Create the project using Maven archetype (this will create 'ecommerce' subdirectory)
mvn archetype:generate -DgroupId=com.example -DartifactId=ecommerce -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false

# Move into the generated project directory
cd ecommerce
```

### 3. Create/Replace pom.xml (Project Object Model)

### This file defines your project's dependencies, build process, and plugins (including Checkstyle for linting).

```
nano pom.xml
```
- Paste the complete pom.xml content and save


## Important for SonarCloud:

#### You'll need to create a project on SonarCloud (https://sonarcloud.io/).
### Once created, SonarCloud will give you an Organization Key and a Project Key. Update the placeholders YOUR_SONARCLOUD_ORGANIZATION_KEY and YOUR_SONARCLOUD_PROJECT_KEY in pom.xml with your actual values.

### 4. Create google_checks.xml (Checkstyle Rules)

### This XML file defines the coding style and formatting rules enforced by Maven Checkstyle.

```
nano google_checks.xml
```
- paste the content of the file and save

### 5. Create the Main Application Class

### This is your simple Spring Boot REST API.

```
mkdir -p src/main/java/com/example/ecommerce
nano src/main/java/com/example/ecommerce/EcommerceApplication.java
```
- Paste the content pf EcommerceApplication.java and save

### 6. Create a Simple Test Class

### A basic test to ensure the Spring context loads

```
mkdir -p src/test/java/com/example/ecommerce
nano src/test/java/com/example/ecommerce/EcommerceApplicationTests.java
```

- Paste the content of EcommerceApplicationTests.java and save

### 7. Local Build and Test (Crucial Verification Step)

### This step confirms your Java setup, Maven configuration, and application code are all working together.

- Ensure you are in your project root: cd ~/Git-Action-Project/spring-ecommerce-ci-cd/ecommerce
- Clean any old build artifacts and perform a fresh install

```
rm -rf target # Manually remove target to clear any potential permission issues
mvn clean install
```
- Expected Output: This should now complete with BUILD SUCCESS. If it fails, review the error message carefully and re-check the previous steps (especially Java environment and pom.xml syntax).

### 8. Create Dockerfile (Instructions for Your Docker Image)

### This file tells Docker how to build an image containing your Spring Boot application.

```
nano Dockerfile
```
- Paste the content of Dockerfile for spring boot application and save

### 9. Local Docker Build & Run (Optional but Recommended)

### Verify your Docker setup works locally before pushing to GitHub.

- Ensure you are in your project root: cd ~/Git-Action-Project/spring-ecommerce-ci-cd/ecommerce
- Build the Docker image

```
sudo docker build -t spring-ecommerce-app .
```

- -t spring-ecommerce-app: Tags your image with a human-readable name for local use.
- .: Specifies the build context, meaning the Dockerfile is in the current directory.
- Run the container:

```
sudo docker run -d -p 8080:8080 --name spring-ecommerce-web spring-ecommerce-app
```

- -d: Runs the container in detached mode (in the background).

- -p 8080:8080: Maps Host Machine's (or VM's) port 8080 to the container's exposed port 8080.

- --name spring-ecommerce-web: Assigns a readable name to your container.

- spring-ecommerce-app: Specifies the image to use.

- VirtualBox Port Forwarding (if applicable): If your Docker container is running inside an Ubuntu VirtualBox VM, for your host machine (your actual computer) to reach it, you need a VirtualBox Port Forwarding rule for the VM: Host Port: 8080 -> Guest Port: 8080.

- Access in your host browser: http://localhost:8080/hello You should see "Hello from Spring Boot E-commerce App!".

- Stop and Remove (after testing):

```
sudo docker stop spring-ecommerce-web
sudo docker rm spring-ecommerce-web
```


## Step Four:
## GitHub Repository Setup & Secrets

### GitHub Actions needs to access your code, Docker Hub, SonarCloud, and your EC2 instance. We use GitHub for code hosting and secure secrets management.

### 1. Create a New Repository on GitHub

### This provides the central location for your code and where GitHub Actions will run.

- Go to https://github.com/new.

- Repository Name: spring-ecommerce-ci-cd (or similar).

- Choose "Public" or "Private." (Public repos get free GitHub Actions minutes and free SonarCloud analysis).

- Click "Create repository."

### 2. Initialize Git Locally and Push Your Files

### This connects your local project to the new GitHub repository.

- Ensure you are in your Spring Boot project directory: cd ~/Git-Action-Project/spring-ecommerce-ci-cd/ecommerce

- Run the following Git commands

```
git init
git add .
git commit -m "Initial commit of Spring Boot E-commerce app with Dockerfile and Checkstyle"
git branch -M main # set main branch name
git remote add origin https://github.com/Git-Action-cicd.git
git push origin -u main
```

### 3. Set Up GitHub Secrets

### GitHub Secrets are encrypted environment variables that you can use in your workflows. They prevent sensitive data (like passwords or SSH keys) from being exposed in your public code or logs.

- Go to your GitHub Repository (https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME).

- Click on Settings (top right tab).

- In the left sidebar, click Secrets and variables > Actions.

## Ensure you are on the "Repository secrets" tab.

- Click "New repository secret" for each of these, providing the exact name and value:

- DOCKER_USERNAME: Your Docker Hub username.

- DOCKER_PASSWORD: Your Docker Hub password.

- EC2_SSH_KEY: The entire content of your EC2 private key file (.pem file).

- How to get it: Open your .pem key file (e.g., my-ec2-key.pem) with a text editor (like nano or VS Code). Copy everything from -----BEGIN RSA PRIVATE KEY----- to -----END RSA PRIVATE KEY----- (including these lines).

- EC2_HOST: The public IP address or public DNS name of your running EC2 instance.

- EC2_USER: The default username for your EC2 instance (e.g., ubuntu for Ubuntu AMIs, ec2-user for Amazon Linux).

- SONAR_TOKEN: Your SonarCloud user token.

- How to get it: Go to SonarCloud (https://sonarcloud.io/), log in, click your profile picture (top right) > My Account > Security. Generate a new token and copy it.


### 4. Set Up GitHub Environment for Manual Approval

## GitHub Environments allow you to define rules for deployments, such as requiring manual approval before a job runs.

- In your GitHub Repository, go to Settings > Environments.

- Click "New environment".

- Name it production: The name must match the environment name used in your workflow file.

- Check "Required reviewers" and add yourself or a team. This means someone (you) must explicitly approve the deployment job before it proceeds.

- Click "Save environment".


## Step Five:
## GitHub Actions Workflow (.github/workflows/main.yml)

### This YAML file describes your entire CI/CD pipeline, defining the sequence of jobs and steps that GitHub Actions will execute.

### 1. Create the Workflow File

### GitHub Actions workflows are defined in YAML files located in the .github/workflows/ directory in your repository.

- Ensure you are in your Spring Boot project directory: cd ~/Git-Action-Project/spring-ecommerce-ci-cd/ecommerce

- Create the workflow directory:

```
mkdir -p .github/workflows
```

- Create the workflow file:

```
nano .github/workflows/main.yml
```

### 2. Paste the Workflow YAML Content and save

### This is the brain of your CI/CD.

### 3. Create sonar-project. properties (for SonarCloud)

### This file tells SonarCloud how to analyze your project. Create it in the root of your ecommerce project (same directory as pom.xml).

```
nano sonar-project.properties
```

- Paste this content of sonar and save

## Important:

- Update YOUR_SONARCLOUD_ORGANIZATION_KEY and YOUR_SONARCLOUD_PROJECT_KEY with the actual values from your SonarCloud account.

- If you're using a private GitHub repository, you might need to adjust SonarCloud settings to allow analysis from private repos.


## Step Six
## Trigger and Monitor the CI/CD Pipeline

### Now, you'll push your code to GitHub, and the magic of CI/CD will begin!

### 1. Commit and Push Your Workflow File and Other Code

### This is the final step to activate your CI/CD pipeline on GitHub.

- Ensure you are in your Spring Boot project directory: cd ~/Git-Action-Project/spring-ecommerce-ci-cd/ecommerce

- Add all new and modified files to Git staging:

```
git add .
git commit -m "Add GitHub Actions CI/CD for Spring Boot with security tools"
git push origin main
```

### 2. Monitor the Workflow Run on GitHub

- Go to your GitHub Repository (https://github.com/YOUR_GITHUB_USERNAME/YOUR_REPO_NAME).

- Click on the "Actions" tab.

- You will see your "Spring Boot CI/CD Pipeline" workflow listed.

- Click on the active run (look for a yellow dot or spinning icon) to see the detailed progress of each job and step. You can click on individual steps to view their logs.

## Observe the Workflow Execution Flow:

### 1. ci_build_lint_test job starts:

- Checks out code.

- Sets up Java 21.

- Runs mvn checkstyle:check (linting). If style violations are found, this job will fail.
- Runs mvn clean install (builds and runs unit tests). If compilation or tests fail, this job will fail.

### 2. security_scan job starts: (if ci_build_lint_test succeeds):

- Checks out code and sets up Java.

- Runs SonarCloud Scan. This will analyze your code for bugs, vulnerabilities, and code smells and push results to your SonarCloud dashboard.

-Runs Gitleaks Scan. This will scan your repository for hardcoded secrets. If any are found, this job will fail.

### 3. manual_approval_for_deploy job starts (if security_scan succeeds and it's a push to main):

- This job will enter a pending state.
You'll see a yellow banner with a "Review deployments" button in the GitHub Actions UI.

- Click this button, review the details (e.g., by looking at the previous job's logs and SonarCloud report), and click "Approve and deploy" to continue.

### 4. docker_build_trivy_push job starts (after approval):

- Checks out code.

- Logs into Docker Hub.

- Builds the Docker image using your Dockerfile.

- Runs Trivy Image Scan on the newly built Docker image. If critical/high vulnerabilities are found, this job will fail.

- Pushes the Docker image to your Docker Hub repository (e.g., YOUR_DOCKER_USERNAME/ecommerce-app:latest).

### 5. deploy_to_ec2 job starts (after docker_build_trivy_push succeeds):

- Uses the appleboy/ssh-action to establish an SSH connection to your EC2 instance.

## Executes the script commands on the EC2 instance:

- Ensures Docker is running.

- Pulls the latest image from Docker Hub.

- Stops and removes any old container named ecommerce-web-container.

- Starts a new container from the fresh image, mapping EC2's port 80 to the container's 8080.


### 6. Verify Deployment on EC2

### Once the deploy_to_ec2 job finishes successfully (all green checkmarks):

- Access the application in your browser:
Open your web browser (on your main computer) and go to:
http://YOUR_EC2_PUBLIC_IP_OR_DNS:8080/hello (Replace with your actual EC2 Public IP or Public DNS).
You should see the message: "Hello from Spring Boot E-commerce App!".

- Note: We mapped EC2's port 80 to the container's 8080. If your EC2 security group only allows port 80, you might need to change -p 80:8080 to -p 80:80 in the workflow's deploy_to_ec2 script and ensure Spring Boot runs on port 80 (by adding server.port=80 to src/main/resources/application.properties in your Spring Boot app). For simplicity and common Spring Boot defaults, we're using 8080 inside the container and mapping to 80 on EC2.

- Verify Docker on EC2 (via SSH):
You can SSH into your EC2 instance and check the Docker components:

```
ssh -i /path/to/your/keypair.pem ubuntu@YOUR_EC2_PUBLIC_IP_OR_DNS
```

### Then run:

```
docker images # Shows all Docker images downloaded on your EC2. Look for your_docker_username/ecommerce-app.
docker ps     # Shows all currently running Docker containers. Look for 'ecommerce-web-container' with 'Up' status and port 0.0.0.0:80->8080/tcp.
```


## Step Seven
## 7. Troubleshooting Tips

- "Command 'java' not found" / "release version 21 not supported":

- Re-run Phase 0 steps carefully. Ensure java -version and javac -version show 21.0.x and echo $JAVA_HOME points to the correct JDK 21 path in the same terminal session where you run Maven.

- Double-check pom.xml <java.version> is 21.
"Failed to delete .../target/checkstyle-result.xml (No such file or directory)":

- Manually delete the target directory: rm -rf target (or sudo rm -rf target) from your project root.

- Ensure your user (Akoson) has full read/write/execute permissions on the project directory:

```
sudo chown -R Akoson:Akoson /home/Akoson/Git-Action-Project/spring-ecommerce-ci-cd/ecommerce
sudo chmod -R u+rwX /home/Akoson/Git-Action-Project/spring-ecommerce-ci-cd/ecommerce
```

## Maven hangs downloading dependencies:

- Check your VM's internet connection. This is usually a network issue.

## GitHub Actions Workflow Failure:
Go to the "Actions" tab in your GitHub repository.

- Click on the failed workflow run (red X).

- Click on the red-highlighted job or step to view its detailed logs. The error message there will be your primary guide.

## Error: missing server host (from appleboy/ssh-action):

- This means your EC2_HOST secret is missing or empty in GitHub. Double-check its value in Settings > Secrets and variables > Actions.

## Error: Authentication failed (from appleboy/ssh-action):

- Your EC2_SSH_KEY secret is incorrect. Ensure you copied the entire content of your .pem file, including BEGIN and END lines, and there are no extra spaces.

- Your EC2_USER secret might be wrong.

- EC2 Security Group might not allow SSH from GitHub's IP ranges (less common, but possible if highly restricted).

## Website not accessible after deployment:

- Check EC2 Security Group: Ensure Port 80 (or 8080 if you didn't map to 80) is open for inbound traffic from 0.0.0.0/0.

- SSH into EC2 and run docker ps to confirm the container is running and its ports are mapped correctly.

- Check EC2's firewall (ufw status). If active, ensure port 80 (or 8080) is allowed.






