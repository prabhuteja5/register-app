pipeline {
	agent { label 'jenkins-agent' }
	tools {
		maven 'Maven3'
		jdk 'Java17'
	}
	environment {
		APP_NAME = "release-app-pipeline"
		RELEASE = "1.0.0"
		DOCKER_USER= "prabhuteja2421"
		DOCKER_PASS = "dockerhub"
		IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
		IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
	}
	stages{
		stage('clean workspace'){
			steps{
				cleanWs()
			}
		}
		stage('checkout from SCM'){
			steps{
				git branch:'main', credentialsId: 'github', url: 'https://github.com/prabhuteja5/register-app'
			}
		}
		stage('build application'){
			steps{
				sh 'mvn clean package'
			}
		}
		stage('test application'){
			steps{
				sh 'mvn test'
			}
		}
		stage('sonarqube analysys'){
			steps {
				script {
					withSonarQubeEnv(credentialsId: 'jenkins-sonarqube-token'){
						sh 'mvn sonar:sonar'
					}
				}
			}
		}
		stage('Quality Gate'){
			steps {
				script {
					waitForQualityGate abortPipeline: false, credentialsId: 'jenkins-sonarqube-token'
				}
			}
		}
		stage('Build & Push Docker Image'){
			steps {
				script {
					docker.withRegistry('',DOCKER_PASS){
						docker_image = docker.build "${IMAGE_NAME}"
					}
					docker.withRegistry('',DOCKER_PASS){
						docker_image.push("${IMAGE_TAG}")
						docker_image.push("latest")
					}
				}
			}
		}
		stage('Trivy Scan'){
			steps {
				script {
					sh ("docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image ${IMAGE_NAME}:latest --no-progress --scanners vuln --exit-code 0 --severity HIGH,CRITICAL --format table")
				}
			}
		}
		stage('Cleanup Artifacts'){
			steps {
				script {
					sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
					sh "docker rmi ${IMAGE_NAME}:latest"
				}
			}
		}
	}
}
