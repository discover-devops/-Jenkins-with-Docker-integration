Here's a step-by-step guide for creating a CI/CD pipeline using Jenkins with Docker integration:

### Prerequisites:
- Jenkins installed and running on your server (ensure Docker plugin is installed).
- Docker installed on the Jenkins server.
- Access to Docker Hub (credentials for pushing images).
- Docker is pre-installed on the target VM for deployment.

---

## **Step 1: Write the Dockerfile**

You have already written the Dockerfile, but just in case, here is the content:

```dockerfile
FROM centos
RUN yum install java -y
RUN mkdir /opt/tomcat/
WORKDIR /opt/tomcat
ADD https://dlcdn.apache.org/tomcat/tomcat-9/v9.0.54/bin/apache-tomcat-9.0.54.tar.gz /opt/tomcat
RUN tar xvfz apache*.tar.gz
RUN mv apache-tomcat-9.0.54/* /opt/tomcat 
EXPOSE 8080
CMD ["/opt/tomcat/bin/catalina.sh", "run"]
```

This Dockerfile sets up an Apache Tomcat server running on CentOS.

---

## **Step 2: Create the CI Job in Jenkins**

### 2.1. Set up Jenkins Freestyle Job to Build and Push Docker Image

1. **Create a New Freestyle Project** in Jenkins:
   - Name: `Build_and_Push_Docker_Image`
   
2. **Configure Source Code Management (SCM)**:
   - If you are using Git, configure your Git repository in this section (if applicable).

3. **Add Build Step**:
   - In the 'Build' section, click **Add build step** → **Execute shell**.
   - In the shell script, add the following commands to build and push your Docker image to Docker Hub:

   ```bash
   # Log in to Docker Hub
   docker login -u discoverdevops -p <Your Docker Hub Password>
   
   # Build Docker image
   docker build -t discoverdevops/myprodapp .
   
   # Tag the image (latest or with version tag)
   docker tag discoverdevops/myprodapp:latest discoverdevops/myprodapp:latest
   
   # Push the image to Docker Hub
   docker push discoverdevops/myprodapp:latest
   ```

4. **Post-build Actions** (Optional):
   - You can set up notifications or triggers (such as Slack or email) if the build succeeds or fails.

5. **Save and Run the Job**:
   - After configuring everything, click on **Save**.
   - You can run this job to ensure that your Docker image gets built and pushed to your Docker Hub repository.

---

## **Step 3: Create the CD Job in Jenkins**

### 3.1. Set up a Freestyle Job to Deploy the Docker Image on the VM

1. **Create a New Freestyle Project**:
   - Name: `Deploy_Docker_Image_to_VM`
   
2. **Configure Build Triggers**:
   - In the "Build Triggers" section, you can choose to trigger the job after the build of the CI job. This ensures that the deployment job will only run after the Docker image is successfully built.

3. **Add Build Step**:
   - Add a build step → **Execute shell** and use SSH to connect to your target VM where Docker is installed.
   - In the shell script, add the following commands:

   ```bash
   # SSH into the target VM (adjust to your SSH details)
   ssh user@target_vm_ip << EOF
   docker pull discoverdevops/myprodapp:latest
   docker stop myprodapp || true
   docker rm myprodapp || true
   docker run -d --name myprodapp -p 8080:8080 discoverdevops/myprodapp:latest
   EOF
   ```

   - This script will pull the latest Docker image from Docker Hub, stop and remove any existing container, and start a new container using the latest image.

4. **Save and Run the Job**:
   - Click on **Save**.
   - You can manually trigger the deployment job or set it to run after the successful completion of the CI job.

---

### **Optional: Secure Jenkins with Docker Credentials**
To avoid exposing Docker Hub credentials in your build script, use Jenkins credentials to store your Docker Hub username and password.

1. In Jenkins, go to **Manage Jenkins** → **Manage Credentials**.
2. Add a new username and password credential for your Docker Hub account.
3. In your build script, replace the login command with the following:
   ```bash
   docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD
   ```
4. Reference these credentials in your job configuration.

---

### **Step 4: Testing the Pipeline**

1. **Test the CI Job**:
   - Run the `Build_and_Push_Docker_Image` job to ensure that it builds the Docker image and pushes it to your Docker Hub repository.

2. **Test the CD Job**:
   - Trigger the `Deploy_Docker_Image_to_VM` job to ensure the Docker image is pulled and deployed on your VM.

---

This tutorial provides a simple yet powerful CI/CD pipeline to automate the build and deployment of Docker applications using Jenkins.
