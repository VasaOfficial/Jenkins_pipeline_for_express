# Guide on making Jenkins ci/cd pipeline for simple express app

<p>This repository contains a comprehensive CI/CD pipeline for automating the build and deployment process using Jenkins. The pipeline includes:</p>

 * Github: For version control
 * Jenkins: For ci/cd pipeline
 * Sonarqube: For secirity and code analysis 
 * Dependency-check: For checking the dependencies
 * Docker: For dockerizing the application
 * AWS ECS: For deployment

![Screenshot_4-min](https://github.com/user-attachments/assets/facb304d-202e-4e26-b3f5-7a127ae4cbe4)

<h3>Few thing that can be improved and added:</h3>

* Terraform for provisioning aws infrastructure
* Test stange
* Trivy for docker image scaning
* Deploying app on k8s, AWS Elastic Beanstalk...

## Setting up Jenkins on a EC2

1. Update System Packages

```
sudo apt-get update
sudo apt-get upgrade -y
```

2. Install Java

```
sudo apt-get install openjdk-11-jdk -y
```

3. Add Jenkins Repository and Install Jenkins

```
wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
sudo sh -c 'echo deb http://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
sudo apt-get update
sudo apt-get install jenkins -y
```

4. Start and Enable Jenkins

```
sudo systemctl start jenkins
sudo systemctl enable jenkins
```

5. Jenkins can now be accessed on the 'http://<your-ec2-instance-ip>:8080'

# Setting up Sonarqube on a EC2

1. Update System Packages

```
sudo apt-get update
sudo apt-get upgrade -y
```

2. Install Java

```
sudo apt-get install openjdk-11-jdk -y
```

3. Install PostgreSQL

```
sudo apt-get install postgresql postgresql-contrib -y
```

4. Set Up PostgreSQL Database for SonarQube

```
sudo -i -u postgres
createuser sonar
createdb sonarqube
psql
ALTER USER sonar WITH ENCRYPTED password 'your_password';
GRANT ALL PRIVILEGES ON DATABASE sonarqube TO sonar;
\q
exit
```

5. Download and Install SonarQube

```
cd /opt
sudo wget https://binaries.sonarsource.com/Distribution/sonarqube/sonarqube-8.9.0.43852.zip
sudo unzip sonarqube-8.9.0.43852.zip
sudo mv sonarqube-8.9.0.43852 sonarqube
```

6. Configure SonarQube

```
sudo nano /opt/sonarqube/conf/sonar.properties
```

```
sonar.jdbc.username=sonar
sonar.jdbc.password=your_password
sonar.jdbc.url=jdbc:postgresql://localhost/sonarqube
```

7. Start SonarQube

```
cd /opt/sonarqube/bin/linux-x86-64
sudo ./sonar.sh start
```

# Make the ECR repository and ECS cluster on the aws

# Making a pipeline

1. Install Sonarqube plugin for jenkins and configure it

2. Install OWASP dependency-check plugin and configure it

3. This is the pipeline script:

```
pipeline {
    agent any

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
        SONAR_URL = 'sonar url here'
        SONAR_TOKEN = 'sonar token here'
        AWS_CREDENTIALS = credentials('aws-credentials-id')
        ECR_REPO = ''
        AWS_REGION = 'eu-central-1'
        APP_NAME = 'expressbasic'
        CLUSTER_NAME = 'ecs cluster name'
        SERVICE_NAME = 'ecs service name'
    }

    stages {
        stage('Git Checkout') {
            steps {
                git branch: 'main', changelog: false, poll: false, url: 'github repo url of the project'
            }
        }

        stage('Sonar Analysis') {
            steps {
                sh '''$SCANNER_HOME/bin/sonar-scanner \
                       -Dsonar.host.url=$SONAR_URL \
                       -Dsonar.login=$SONAR_TOKEN \
                       -Dsonar.projectName='project name here' \
                       -Dsonar.projectKey='project name here' \
                       -Dsonar.sources=.'''
            }
        }

        stage('OWASP Dependency Check') {
            steps {
               dependencyCheck additionalArguments: ' --scan ./', odcInstallation: 'DP'
               dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $APP_NAME:latest ."
                    sh "docker tag $APP_NAME:latest $ECR_REPO:latest"
                }
            }
        }

        stage('Push to ECR') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials-id']]) {
                        sh """
                            aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                            docker push $ECR_REPO:latest
                        """
                    }
                }
            }
        }

        stage('Deploy to ECS') {
            steps {
                script {
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials-id']]) {
                        sh """
                            aws ecs update-service \
                            --cluster $CLUSTER_NAME \
                            --service $SERVICE_NAME \
                            --force-new-deployment \
                            --region $AWS_REGION
                        """
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                cleanWs()
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            mail to: 'team@example.com',
                 subject: "SUCCESS: ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                 body: "Good news! The build ${env.BUILD_NUMBER} succeeded."
        }
        failure {
            mail to: 'team@example.com',
                 subject: "FAILURE: ${env.JOB_NAME} ${env.BUILD_NUMBER}",
                 body: "Unfortunately, the build ${env.BUILD_NUMBER} failed. Please check the Jenkins logs for details."
        }
    }
}
```
