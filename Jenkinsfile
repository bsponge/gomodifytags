def builderImage = "bsponge/builder:1.0.5"
def testerImage = "bsponge/tester:1.0.5"
def deployerImage = "bsponge/deployer:1.0.5"
def deploymentImage = "deployment-image"

pipeline {
	agent any

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
		}
	post {
		always {
			sh "docker rm -f builder"
				sh "rm *.log"
				sh "docker rm -f tester"
				sh "docker rm -f cloner"
				sh "docker rm -f test-deployment"
				sh "docker rmi ${builderImage}"
				sh "docker rmi ${testerImage}"
				sh "docker rmi test-deploy"

				archiveArtifacts artifacts: 'pipeline.log' 
		}
	}
}
