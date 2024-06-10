pipeline{

    agent any
/*
	tools {
        maven "maven3"
    }
*/
    environment {
        registry= "mohamedosama113/vproapp"
        registryCredential= "dockerhub"
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

        /*stage('CODE ANALYSIS with SONARQUBE') {

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

                timeout(time: 10, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }*/
        stage("Build Docker APP"){
            steps{
                script{

                    dockerImage = docker.build(registry + "v:${env.BUILD_ID}")
                }
                        }
    
        }
 
         stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Log in to Docker Hub and push the image
                    docker.withRegistry('', registryCredential) {
                        dockerImage.push("v${env.BUILD_ID}")
                        dockerImage.push('latest')
                    }
                }
            }
        }
          /*      stage('Clean Up Unused Docker Images') {
            steps {
                script {
                    // Remove dangling and unused Docker images
                    sh 'docker rmi $registry:v$BUILD_ID'
                }
            }
        }*/ 
         
        stage("Kuber Deploy"){
            agent{label 'KUBEMASTER'}
            steps{
                sh"kubectl create namespace prod"
                sh"helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${registry}:v${BUILD_ID} --namespace prod"
            }
            post{
                always{
                    echo "====++++always++++===="
                }
                success{
                    echo "====++++Kuber Deploy executed successfully++++===="
                }
                failure{
                    echo "====++++Kuber Deploy execution failed++++===="
                }
        
            }
        }
    }

}
