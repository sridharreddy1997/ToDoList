pipeline {
    agent {label 'Build-Server'}  // Use any available agent to run the pipeline
environment {
        DOCKER_REPO = 'bhaskardanda78/boardgame'
        DOCKER_IMAGE_TAG = "${env.BUILD_NUMBER}"
        DOCKER_IMAGE = "${env.DOCKER_IMAGE_NAME}:${env.DOCKER_IMAGE_TAG}"
        DOCKER_REGISTRY = 'https://index.docker.io/v1/'
        DOCKER_LATEST = "${env.DOCKER_IMAGE_NAME}:latest"
        KUBECONFIG = "/root/cluster-kubeconfig.yaml"
        SONARQUBE = 'SonarQube' // Name of the SonarQube installation in Jenkins
    }
    stages {
        stage('Checkout') {
            steps {
                // Checkout the code from a Git repository
                git url: 'https://github.com/BhaskarDanda/Boardgame.git', branch: 'main'
            }
        }
stage('Build') {
             steps {
                 sh '''

cd $WORKSPACE/
ls -lrt 
/root/maven/bin/mvn clean package
ls -lrt
 '''
             }
         }
        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    dockerImage = docker.build("${DOCKER_REPO}:${BUILD_NUMBER}")
                }
            }
        }
        
        stage('Tag Docker Image') {
            steps {
                script {
                    // Tag the image with "latest"
                    sh "docker tag ${DOCKER_REPO}:${BUILD_NUMBER} ${DOCKER_REPO}:latest"
                }
            }
        }
        
        stage('Push Docker Images') {
            steps {
                script {
                    docker.withRegistry('', 'Docker_Cred') {
                        // Push the build number tagged image
                        dockerImage.push("${BUILD_NUMBER}")
                        // Push the "latest" tagged image
                        dockerImage.push("latest")
                    }
                }
            }
        }
stage('deployment') {
        steps {
            script {
                sh "kubectl apply -f deployment-service.yaml"
            }
        }
    }
stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQube'
                    withSonarQubeEnv('SonarQube') {
                    sh """${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=Boardgame -Dsonar.projectName=Boardgame -Dsonar.projectVersion=1.0"""
                    }
                }
            }
        }
        // stage('Quality Gate') {
        //     steps {
        //         script {
        //             // timeout(time: 1, unit: 'HOURS') {
        //                 waitForQualityGate abortPipeline: true
        //             // }
        //         }
        //     }
        // }
    }
post {
        always {
            // Actions that are always performed after the pipeline
            echo 'Pipeline completed.'
            script {
                sh "docker rmi ${DOCKER_REPO}:${BUILD_NUMBER} || true"
                sh "docker rmi ${DOCKER_REPO}:latest || true"
            }
        }
        success {
            echo 'Pipeline succeeded.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}