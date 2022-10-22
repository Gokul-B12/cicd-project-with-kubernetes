pipeline {

    agent any

	tools {
        maven "Maven3"
    }

    environment {
        registry = "gokulb12/kube-cicdimg"
        registryCredential = "dockerhub"
        ARTVERSION = "${env.BUILD_ID}"
    }

    stages{

        
        stage('BUILD'){
            steps {
                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo 'Now Archiving...'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
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
                scannerHome = tool 'sonar4.7'
            }

            steps {
                withSonarQubeEnv('sonar') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                   -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Bluid App Image')
        {
            steps{
                script {
                    dockerImage = docker.build registry + ":v$BUILD_NUMBER"
                }
            }
        }
        stage("Upload image") {
            steps{
                script{
                    docker.withRegistry("", registryCredential){
                        dockerImage.push("v$BUILD_NUMBER")
                        dockerImage.push("latest")
                    }
                    
                }
            }
        }

        stage("Remove unused docker images") {
            steps{
                sh "docker rmi $registry:v$BUILD_NUMBER"
            }
        }

        stage("Deploying img in K8s"){
            agent{label 'kops'}
            steps{
                sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set tomappimg=${registry}:v${BUILD_NUMBER} --namespace prod"
            }
        }
    




    }


}
