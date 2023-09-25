pipeline {
	agent { label 'jenkins-agent' }
	tools {
		maven 'Maven3'
		jdk 'Java17'
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
				sh "mvn clean package"
			}
		}
		stage('test application'){
			steps{
				sh "mvn test"
			}
		}
	}
}
