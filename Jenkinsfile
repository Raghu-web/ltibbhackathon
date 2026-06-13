pipeline {
    agent any
    environment {
        scannerHome = tool 'mysonar';
    }
    stages {
        stage('Hello') {
            steps {
                git 'https://github.com/Raghu-web/ltibbhackathon.git'
            }
        }
        stage('CQA') {
            steps {
                withSonarQubeEnv('mysonar') {
                    sh "${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=myproject"
                }
            }
        }
        stage('Quality gates') {
            steps {
                waitForQualityGate abortPipeline: false, credentialsId: 'mysonarqube'
            }
        }
        stage('Docker images') {
            steps {
                sh 'docker build -t dbimage database'
                sh 'docker build -t appimage .'
            }
        }
        stage('image scan') {
            steps {
                sh 'trivy image dbimage'
                sh 'trivy image appimage'
            }
        }
        stage('image tagging') {
            steps {
                sh 'docker tag dbimage raghuram0024/blooddonationapp:dbimage'
                sh 'docker tag appimage raghuram0024/blooddonationapp:appimage'
            }
        }
        stage('push') {
            steps {
                script {
                    withDockerRegistry(credentialsId: 'dockerhub-cred') {
                        sh 'docker push raghuram0024/blooddonationapp:dbimage'
                        sh 'docker push raghuram0024/blooddonationapp:appimage'
                    }
                }
            }
        }
        stage('Deploy') {
            steps {
                sh 'docker run -d --name mysqldb -p 3306:3306 raghuram0024/blooddonationapp:dbimage'
                sh 'docker run -d --name app-container -p 2222:80 --link mysqldb:mysqldbcont raghuram0024/blooddonationapp:appimage'
            }
        }
    }
}
