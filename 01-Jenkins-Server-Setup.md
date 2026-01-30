# Jenkins Server Setup in Linux VM #

## Step - 1 : Create Linux VM ##

1) Create Ubuntu VM using AWS EC2 (t2.medium) <br/>
2) Enable 8080 Port Number in Security Group Inbound Rules
3) Connect to VM using MobaXterm

## Step-2 : Instal Java ##

```
sudo apt update
sudo apt install fontconfig openjdk-17-jre
java -version
```

## Step-3 : Install Jenkins ##
```
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
  https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update
sudo apt-get install jenkins
```

## Step-4 : Start Jenkins ## 

```
sudo systemctl enable jenkins
sudo systemctl start jenkins
```

## Step-5 : Verify Jenkins ##

```
sudo systemctl status jenkins
```
	
## Step-6 : Open jenkins server in browser using VM public ip ##

```
http://public-ip:8080/
```

## Step-7 : Copy jenkins admin pwd ##
```
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```
	   
## Step-8 : Create Admin Account & Install Required Plugins in Jenkins ##



## DEMO Pipeline

Note - must used the job as a pipeline not free style okk
```
pipeline {
    agent any

    environment {
        REPO_URL = "https://github.com/KunalMali-The-Clever-Programmer/SpringBoot_Deploying-_To_AWS_EC2-.git"
        APP_NAME = "springboot-app"
        DOCKER_IMAGE = "kunalmali/${APP_NAME}:latest"
        SONARQUBE = "SonarQube" // Name of SonarQube server configured in Jenkins
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Cloning repository..."
                git url: "${REPO_URL}", branch: "main"
            }
        }

        stage('Maven Build') {
            steps {
                echo "Building with Maven..."
                sh "mvn clean package -DskipTests"
            }
        }

        stage('SonarQube Analysis') {
            steps {
                echo "Running SonarQube scan..."
                withSonarQubeEnv("${SONARQUBE}") {
                    sh "mvn sonar:sonar"
                }
            }
        }

        stage('Docker Build') {
            steps {
                echo "Building Docker image..."
                sh """
                docker build -t ${DOCKER_IMAGE} .
                """
            }
        }

        stage('Docker Push') {
            steps {
                echo "Pushing Docker image..."
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh """
                    echo $PASS | docker login -u $USER --password-stdin
                    docker push ${DOCKER_IMAGE}
                    """
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                echo "Deploying Docker container..."
                // Assuming Docker is installed on the EC2 itself
                sh """
                docker stop ${APP_NAME} || true
                docker rm ${APP_NAME} || true
                docker run -d --name ${APP_NAME} -p 8080:8080 ${DOCKER_IMAGE}
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully 🚀"
        }
        failure {
            echo "Pipeline failed ❌"
        }
    }
}

```
