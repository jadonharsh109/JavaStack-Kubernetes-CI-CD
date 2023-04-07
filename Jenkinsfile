pipeline {

    agent any

    tools {
        maven "MAVEN3"
        jdk "OracleJDK8"
        }

    environment {
        DOCKERHUB_REPO = "jadonharsh/vprofileapp"
        DOCKERHUB_CRED = "dockerhub"
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



        stage('DOCKER BUILD'){
            steps{
                script {
                    dockerImage = docker.build DOCKERHUB_REPO + ":$BUILD_NUMBER"
                }
            }
        }

        stage('DEPLOY BUILD') {
            steps{
                script {
                    docker.withRegistry( '', DOCKERHUB_CRED ) {
                    dockerImage.push("$BUILD_NUMBER")
                    dockerImage.push('latest')
                    }
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
                    sh "helm upgrade --install --force vproifle-stack applicationcharts --set image=${registry}:${BUILD_NUMBER} --namespace prod"
            }
        }




    }


}