pipeline {

    agent any

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

        stage('CODE ANALYSIS with SONARQUBE') {

            environment {
                scannerHome = tool 'mysonarscanner4'
            }

            steps {
                withSonarQubeEnv('sonar-pro') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile-repo \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
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