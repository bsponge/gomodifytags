def builderImage = "bsponge/builder:1.0.5"
def testerImage = "bsponge/tester:1.0.5"
def deployerImage = "bsponge/deployer:1.0.5"
def deploymentImage = "bsponge/deployment-image"

pipeline {
	agent any
	parameters {
		string(name: 'version', defaultValue: 'latest', description: '')
		booleanParam(name: 'promote', defaultValue: 'true', description: '')
	}

	environment {
		DOCKERHUB_CREDENTIALS=credentials("docker-hub-creds")
	}

	stages {
		stage('prepare') {
			steps {
				sh "docker volume create output"
				sh "docker volume create input"
				sh "docker run --name cloner -dit -v input:/input alpine:latest"
				sh "docker cp . cloner:/input"
				sh "docker rm -f cloner"
			}
		}
		stage('build') {
			steps {
				sh "docker run --name builder -v input:/input -v output:/output ${builderImage}"
			}
		}
		stage('test') {
			steps {
				sh "docker run --name tester -v input:/input ${testerImage}"
			}
		}
		stage('create artifacts') {
			steps {
				sh "docker logs builder >> pipeline-${env.version}.log"
				sh "docker logs tester >> pipeline-${env.version}.log"
			}
		}
		stage('deploy') {
			steps {
				sh "docker build -t ${deploymentImage}:${env.version} -f Dockerfile-deploy ."
			}
		}
		stage('test deploy') {
			steps {
				sh "docker build -t test-deploy -f Dockerfile-test-deploy --build-arg image=${deploymentImage}:${env.version} ."
				sh "docker run --name test-deployment test-deploy"
				sh "docker logs test-deployment >> testoutput.log"
				sh "diff testoutput.log jenkins/expected.go"
			}
		}
		stage('publish') {
			steps {
				script {
					if (env.promote == "true") {
						sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
						sh "docker push ${deploymentImage}:${env.version}"
					} else {
						sh "echo NOT PUBLISHING"
					}
				}
			}
		}
	}
	post {
		always {
			archiveArtifacts artifacts: '*.log' 
			sh "rm *.log"

			sh "docker rm -f \$(docker ps -a -q)"
			sh "docker system prune -af"
			sh "docker rm \$(docker ps -a -q)"

			sh "docker logout"
		}
	}
}
