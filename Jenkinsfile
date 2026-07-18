pipeline {

    agent any 
  

    environment {
    IMAGE_IMAGE = "faisalkhan35/spring-boot-app"
    TAG = "${BUILD_NUMBER}"
    NEXUS_CREDS = credentials('nexus-creds')
    DOCKERHUB_CREDS = credentials('dockerhub-creds')
    }

    tools {
        maven 'Maven17'
    }

    stages {
  
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

        stage('Nexus Upload') {
            steps {
                sh 'curl -v -u $NEXUS_CREDS_USR:$NEXUS_CREDS_PSW --upload-file target/Spring-Boot-application.-0.0.1-SNAPSHOT.jar http://40.81.235.72:8081/repository/maven-snapshots/com/example/Spring-Boot-application./0.0.1-SNAPSHOT/Spring-Boot-application.-0.0.1-SNAPSHOT.jar'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t ${IMAGE_IMAGE}:${TAG} .'
            }
        }

        stage('Docker Hub Login') {
            steps {
                sh 'echo $DOCKERHUB_CREDS_PSW | docker login -u $DOCKERHUB_CREDS_USR --password-stdin'
            }
        }

        stage('Docker Hub Push') {
            steps {
                sh 'docker push ${IMAGE_IMAGE}:${TAG}'
            }
        }

        stage('AKS Deploy') {
            steps {
                sh '''
                kubectl set image deployment/spring-boot-app spring-boot-app=${IMAGE_IMAGE}:${TAG}
                '''
            }
        }


        stage('Health Check') {
            steps {
                sh '''
                echo "Checking Application Health Check"
                sleep 15
                curl --fail http://4.224.190.126/ || exit 1
                echo "Health Check is Failed"
                '''
            }
        }
    }
 
    post {

        success {
            sh 'echo "Application is Successfully Deployed"'
        }

        failure {
            sh '''
            echo "Application Deployment is Failed rollback is initiated"
            kubectl rollout undo deployment/spring-boot-app
            '''
        }
    }
}
