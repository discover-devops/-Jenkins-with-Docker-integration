### Use Jenkins plugins to automate the CI and CD jobs more efficiently. Here’s how you can achieve that:

### **1. Automate the CI Job Using Jenkins Plugins**

For building and pushing Docker images, Jenkins provides a **Docker Pipeline Plugin** that integrates directly with Docker. This removes the need for manual shell scripts.

#### **Step-by-Step Setup for the CI Job:**

1. **Install Docker Plugins:**
   - Go to **Manage Jenkins** → **Manage Plugins** → **Available** and search for the following plugins:
     - **Docker Pipeline Plugin** (for Docker integration)
     - **Pipeline** (if not already installed)

2. **Create a New Pipeline Job:**
   - Instead of a Freestyle job, create a **Pipeline** job in Jenkins.
   - Name it `Build_and_Push_Docker_Image`.

3. **Configure the Pipeline Script:**
   - In the **Pipeline** section, choose **Pipeline script** and use the following Jenkins pipeline code:

   ```groovy
   pipeline {
       agent any
       environment {
           DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials-id') // Add your Docker Hub credentials in Jenkins
       }
       stages {
           stage('Checkout Code') {
               steps {
                   git 'https://your-git-repository-url'  // Set your repo URL
               }
           }
           stage('Build Docker Image') {
               steps {
                   script {
                       docker.build("discoverdevops/myprodapp")
                   }
               }
           }
           stage('Push Docker Image to Docker Hub') {
               steps {
                   script {
                       docker.withRegistry('https://index.docker.io/v1/', 'DOCKER_HUB_CREDENTIALS') {
                           def app = docker.build("discoverdevops/myprodapp")
                           app.push('latest')
                       }
                   }
               }
           }
       }
   }
   ```

   - **Explanation**:
     - **DOCKER_HUB_CREDENTIALS**: This pulls your Docker Hub credentials from Jenkins Credentials Manager (you need to add the credentials first).
     - **docker.build**: Builds the Docker image using the `Dockerfile` in your repository.
     - **docker.withRegistry**: Logs into Docker Hub and pushes the image to your repository.

4. **Save and Run the Job:**
   - After configuring the job, save it.
   - Run the job to verify that the image is built and pushed to Docker Hub.

---

### **2. Automate the CD Job Using Jenkins Plugins**

For deploying the Docker image on a remote VM, you can use the **SSH Pipeline Steps Plugin**, which allows you to run commands on remote servers using SSH.

#### **Step-by-Step Setup for the CD Job:**

1. **Install SSH Pipeline Steps Plugin:**
   - Go to **Manage Jenkins** → **Manage Plugins** → **Available** and install the **SSH Pipeline Steps Plugin**.

2. **Create a New Pipeline Job:**
   - Create another **Pipeline** job in Jenkins.
   - Name it `Deploy_Docker_Image_to_VM`.

3. **Add SSH Credentials for Your VM:**
   - Go to **Manage Jenkins** → **Manage Credentials** → Add **SSH Username with private key** credentials for your VM.
   - Note down the ID of the credentials you just created (this will be used in the pipeline script).

4. **Configure the Pipeline Script for CD Job:**
   - In the **Pipeline** section, add the following pipeline code:

   ```groovy
   pipeline {
       agent any
       environment {
           VM_CREDENTIALS = credentials('vm-ssh-credentials-id') // SSH credentials for your VM
       }
       stages {
           stage('Deploy Docker Image on VM') {
               steps {
                   sshagent(['VM_CREDENTIALS']) {
                       sh '''
                           ssh -o StrictHostKeyChecking=no user@target_vm_ip '
                           docker pull discoverdevops/myprodapp:latest &&
                           docker stop myprodapp || true &&
                           docker rm myprodapp || true &&
                           docker run -d --name myprodapp -p 8080:8080 discoverdevops/myprodapp:latest'
                       '''
                   }
               }
           }
       }
   }
   ```

   - **Explanation**:
     - The `sshagent` block ensures the VM credentials are used to SSH into the target VM.
     - The `sh` block executes the Docker commands to pull, stop, and run the latest image on the VM.

5. **Save and Run the Job:**
   - Save the configuration and run the job to deploy the Docker image on the VM.

---

### **Bonus: Automating the Trigger between CI and CD Jobs**

To trigger the CD job automatically after the CI job completes, you can use the **Build Trigger Plugin** or set up a pipeline that combines both CI and CD steps.

1. **Set Up Trigger:**
   - In the **Deploy_Docker_Image_to_VM** job, go to the **Build Triggers** section and select **Build after other projects are built**.
   - Enter the name of the CI job `Build_and_Push_Docker_Image`.

2. **Pipeline for End-to-End Automation:**
   - You can combine the CI and CD steps into a single Jenkins pipeline:

   ```groovy
   pipeline {
       agent any
       environment {
           DOCKER_HUB_CREDENTIALS = credentials('docker-hub-credentials-id')
           VM_CREDENTIALS = credentials('vm-ssh-credentials-id')
       }
       stages {
           stage('Checkout Code') {
               steps {
                   git 'https://your-git-repository-url'
               }
           }
           stage('Build Docker Image') {
               steps {
                   script {
                       docker.build("discoverdevops/myprodapp")
                   }
               }
           }
           stage('Push Docker Image to Docker Hub') {
               steps {
                   script {
                       docker.withRegistry('https://index.docker.io/v1/', 'DOCKER_HUB_CREDENTIALS') {
                           def app = docker.build("discoverdevops/myprodapp")
                           app.push('latest')
                       }
                   }
               }
           }
           stage('Deploy Docker Image on VM') {
               steps {
                   sshagent(['VM_CREDENTIALS']) {
                       sh '''
                           ssh -o StrictHostKeyChecking=no user@target_vm_ip '
                           docker pull discoverdevops/myprodapp:latest &&
                           docker stop myprodapp || true &&
                           docker rm myprodapp || true &&
                           docker run -d --name myprodapp -p 8080:8080 discoverdevops/myprodapp:latest'
                       '''
                   }
               }
           }
       }
   }
   ```

This pipeline will handle both the CI (build and push Docker image) and CD (deploy image on VM) in a seamless flow.

---

By using these plugins, you can avoid writing manual shell scripts and leverage Jenkins’ robust automation capabilities for Docker integration.
