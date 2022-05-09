def builderImage = "bsponge/builder:1.0.5"
def testerImage = "bsponge/tester:1.0.5"

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
        stage('cleanup') {
            steps {
                sh "docker logs builder >> pipeline.log"
                sh "docker logs tester >> pipeline.log"
                sh "docker run --name copier -dit -v output:/output alpine:latest"
		sh "ls"
                sh "docker cp copier:/output/gomodifytags gomodifytags"
		sh "ls"
            }
        }
    }
    post {
        always {
            sh "docker rm -f builder"
            sh "docker rm -f tester"
            sh "docker rm -f copier"
	    sh "docker rm -f cloner"
            sh "docker rmi ${builderImage}"
	    sh "docker rmi ${testerImage}"
	    sh "ls"

	    archiveArtifacts artifacts: 'gomodifytags'
	    archiveArtifacts artifacts: 'pipeline.log' 
	}
    }
}
