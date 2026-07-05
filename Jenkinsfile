pipeline {
 
    agent any

    environment {
        IMAGE_NAME = "faisalkhan35/spring-boot-app:latest"
        DOCKERHUB_CREDS = credentials('dockerhub-creds')
        NEXUS_CREDS = credentials('nexus-creds')

    }

    tools {
        maven 'Maven17'
    }
    
    stages {

        stage('GitHub Clone') {
            steps {
                checkout scm
            }
        }
   
        stage('Maven Build') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('SonarQube') {
            steps {
                withSonarQubeEnv('sonar-scanners') {
                    sh 'mvn sonar:sonar'
                }
            }
        }

        stage('Nexus') {
            steps {
                sh 'curl -u $NEXUS_CREDS_USR:NEXUS_CREDS_PSW --upload-file target/Spring-Boot-application.-0.0.1-SNAPSHOT.jar http://4.213.227.29:8081/#browse/browse:maven-central'
            }
        }

        stage('Docker build') {
            steps {
                sh 'docker build -t ${IMAGE_NAME} .'
            }
        }


        stage('DockerHub Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin'
            }
        }

        stage('Docker Hub push') {
            steps {
                sh 'docker push ${IMAGE_NAME}'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl create deployment spring-app --image=${IMAGE_NAME}'
            }
        }
    }
}
