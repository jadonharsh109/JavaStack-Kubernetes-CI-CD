pipeline {

    agent any

    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
        }

    environment {
        DOCKERHUB_REPO = "jadonharsh/vprofileapp"
        DOCKERHUB_CRED = "dockerhubpass"
        VOL_ID = "vol-0c90386274cb9ae63"
    }

    stages{
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
        }

        stage('UNIT TEST'){
            steps {
                sh 'mvn test'
            }
        }

        stage('INTEGRATION TEST'){
            steps {
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage ('CODE ANALYSIS WITH CHECKSTYLE'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
            post {
                success {
                    echo 'Generated Analysis Result'
                }
            }
        }


        stage('Build Docker Image'){
            steps{
                
            sh 'docker build -t ${DOCKERHUB_REPO}:${BUILD_NUMBER} .'
            }
        }

        stage('Push Docker Image'){
            steps{
                
            withCredentials([string(credentialsId: 'dockerhubpass', variable: 'DOKCER_HUB_PASSWORD')]) {
            sh "docker login -u jadonharsh -p ${DOKCER_HUB_PASSWORD}"
            sh 'docker push ${DOCKERHUB_REPO}:${BUILD_NUMBER}'
            }
        }
        }

        stage('REMOVE UNUSED DOCKER IMAGE') {
            steps{
                sh "docker rmi $DOCKERHUB_REPO:$BUILD_NUMBER"
            }
        }

        stage('Kubernetes Deploy') {
	        agent { label 'KOPS' }
                steps {
                    sh "helm upgrade --install --force javastackapp applicationcharts --set image=${DOCKERHUB_REPO}:${BUILD_NUMBER} volumeID=${VOL_ID}"
            }
        }




    }


}