def builderImage = "bsponge/builder:1.0.5"
def testerImage = "bsponge/tester:1.0.5"
def deployerImage = "bsponge/deployer:1.0.5"
def deploymentImage = "bsponge/deployment-image"

pipeline {
	agent any

	environment {
		DOCKERHUB_CREDENTIALS=credentials("docker-hub-creds")
		env.GIT_COMMIT = sh(script: "git rev-parse HEAD", returnStdout: true).trim()
		deploymentImage += ":"
		deploymentImage += env.GIT_COMMIT
	}

	stages {
		stage('prepare') {
			steps {
				sh "docker volume create output"
					sh "docker volume create input"
					sh "docker run --name cloner -dit -v input:/input alpine:latest"
					sh "docker cp . cloner:/input"
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
				sh "docker logs builder >> pipeline.log"
					sh "docker logs tester >> pipeline.log"
			}
		}
		stage('deploy') {
			steps {
				sh "docker build -t ${deploymentImage} -f Dockerfile-deploy ."
			}
		}
		stage('test deploy') {
			steps {
				sh "docker build -t test-deploy -f Dockerfile-test-deploy --build-arg image=${deploymentImage} ."
					sh "docker run --name test-deployment test-deploy"
					sh "docker logs test-deployment >> testoutput.log"
					sh "diff testoutput.log jenkins/expected.go"
			}
		}
		stage('publish') {
			steps {
				sh "echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin"
					sh "docker push ${deploymentImage}"
			}
		}
	}
	post {
		always {
			archiveArtifacts artifacts: 'pipeline.log' 
			sh "rm *.log"

			sh "docker rm -vf \$(docker ps -aq)"
			sh "docker rmi -f \$(docker images -aq)"

			sh "docker logout"
		}
	}
}
