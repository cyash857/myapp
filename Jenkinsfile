pipeline {
    agent any

    environment {
        DATE = new Date().format('yy.M')
        TAG = "${DATE}.${BUILD_NUMBER}"
    }
    stages {
        stage ('Build') {
            steps {
                sh 'mvn clean package'
            }
        }
         stage ('Test') {
	     steps {
	         sh 'mvn surefire:test'
	     }	     
        }
	 stage('SonarQube analysis') {
             steps {
                withSonarQubeEnv('sonarqube') {
                    sh 'mvn clean verify sonar:sonar -Dsonar.projectKey=cyash857 -Dsonar.host.url=http://localhost:9000'
                }
             }
        }
	 stage ('OWASP Dependency Check') {
	     steps {
		 sh 'mvn clean install org.owasp:dependency-check-maven:check -Ddependency-check-format=XML'
		 sh 'aws s3 cp target/dependency-check-report.html s3://pocbucketreport/owaspReport/'    
             }
        }
        stage('Docker Build') {
            steps {
                script {
                    docker.build("default-docker-local/myapp:${TAG}")
                }
            }
        }
	stage('Pushing Docker Image to Jfrog Artifactory') {
            steps {
                script {
                    docker.withRegistry('https://cyash857.jfrog.io/', 'artifactory-credential') {
                        docker.image("default-docker-local/myapp:${TAG}").push()
                        docker.image("default-docker-local/myapp:${TAG}").push("latest")
                    }
                }
            }
        }
        stage('Deploy'){
            steps {
                sh "docker stop myapp | true"
                sh "docker rm myapp | true"
                sh "docker run --name myapp -d -p 80:8080 cyash857.jfrog.io/default-docker-local/myapp:${TAG}"
            }
        }
    }
}
