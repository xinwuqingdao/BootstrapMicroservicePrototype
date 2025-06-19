
🚀 Microservices Prototype Tutorial: Spring Boot + Docker + Jenkins + Kubernetes (macOS)

Environments:
1. macOS 12.7.6
2. Git installed

🧩 Step 0: Install Rancher Desktop with Kubernetes
Rancher Desktop lets you run a local Kubernetes cluster with Docker container runtime.

📥 1. Download Rancher Desktop
Visit: https://rancherdesktop.io

Click “Download for macOS (Intel or Apple Silicon)” depending on your chip.

💿 2. Install the App
Open the downloaded .dmg file and drag Rancher Desktop to Applications.

Launch it.

⚙️ 3. Configure Kubernetes
On first launch:

Container Runtime: Select dockerd (moby)

Kubernetes: Enable and choose a version (default is fine)

Rancher Desktop will install kubectl and set the default Kubernetes context.

🧪 4. Verify Installation
Open a terminal:

	kubectl version --client
	kubectl get nodes
	
You should see a local Kubernetes node running.

📝 Notes
Rancher Desktop installs kubectl to /usr/local/bin/kubectl

You do not need Minikube or Docker Desktop for Kubernetes anymore (Rancher handles it)

✅ Step 1: Install Docker Desktop on macOS (This step is optional)
(Optional if you're using Rancher Desktop's dockerd — but needed for broader Docker support)

Download from: https://www.docker.com/products/docker-desktop

Install and run Docker Desktop

Verify:

	docker --version
	docker run hello-world
🧠 If you see: Cannot connect to the Docker daemon, make sure Docker is running.

if you are using the dockerrd from Rancher Desktop, please do the following:
export DOCKER_HOST=unix:///Users/xinwu/.rd/docker.sock
docker run hello-world

In Jenkinsfile, Also we need to add this DOCKER_HOST variable explicitly if Jenkins server and Rancher Desktop are co existing at the host machine.

Or make it permanent:
nano ~/.bash_profile
export DOCKER_HOST=unix:///Users/xinwu/.rd/docker.sock
🧪 Verify Socket Path Exists
You can confirm Rancher’s Docker daemon is live:
	ls -l /Users/xinwu/.rd/docker.sock
If the file exists, Rancher's Docker is running.

✅ Step 2: Install Java 17 or using dmg file to install

curl -s "https://get.sdkman.io" | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install java 17.0.10-tem
java -version

🛠️ Option 2: Install Oracle’s JDK 17 from DMG

✅ Step 3: Create Spring Boot Project
Visit: https://start.spring.io

Use:

Project: Maven

Language: Java

Dependencies: Spring Web

Unzip and open the project.

Sample HelloController.java

	@RestController
	public class HelloController {
    		@GetMapping("/hello")
    		public String sayHello() {
        		return "Hello, World";
    		}
	}

Build the project:
	./mvnw clean package


✅ Step 4: Create Dockerfile
Place in the project root:
	FROM openjdk:17-jdk-slim
	VOLUME /tmp
	ARG JAR_FILE=target/*.jar
	COPY ${JAR_FILE} app.jar
	ENTRYPOINT ["java", "-jar", "/app.jar"]

✅ Step 5: Install Jenkins on macOS
	Option: Install via Jenkins WAR (Manual, Advanced)
	Download Jenkins WAR
	https://www.jenkins.io/download/

	java -jar jenkins.war
	Access at: http://localhost:8080
	
	get secret for login Jenkins server: 
     cat /Users/$(whoami)/.jenkins/secrets/initialAdminPassword
     
     Follow setup wizard → Install recommended plugins → Create admin user
     
✅ Step 6: Add Jenkinsfile and Kubernetes Config
Jenkinsfile (in project root)
k8s/deployment.yaml

✅ Step 7: Run Jenkins Pipeline
Open Jenkins → New Item → Name: hello-pipeline → Type: Pipeline

Choose:

Pipeline script from SCM

SCM: Git

URL: https://github.com/your-org/hello-springboot.git

Branch: */main

Script Path: Jenkinsfile

Click Build Now

✅ Step 8: Deploy to Kubernetes with Rancher Desktop
Ensure Rancher Desktop is running with Kubernetes enabled

Check context:

kubectl config current-context
Deploy:
kubectl apply -f k8s/deployment.yaml
kubectl get pods
kubectl get svc


Access the service:
http://localhost:30080/hello


 🧹 Recap Project Structure
 
 hello-springboot/
├── Jenkinsfile
├── Dockerfile
├── pom.xml
├── k8s/
│   └── deployment.yaml
└── src/
    └── main/
        └── java/com/example/hello/
            ├── HelloApplication.java
            └── HelloController.java
     

     

