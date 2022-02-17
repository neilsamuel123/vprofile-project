pipeline{

    agent any

    environment {
        registry = "practicechill/tomcat-imran"
        registryCredential = 'dockerhub'
    }

    stages{

        stage('BUILD'){
            steps{
                sh 'mvn clean install -DskipTests'
            }
            post{
                success{
                    echo 'Archiving now ..'
                    archiveArtifacts artifacts: '**/target/*.war'
                }
            }
        }


        stage('Unit Test'){
            steps{
                sh 'mvn test'
            }
        }

        stage('Integration Test'){
            steps{
                sh 'mvn verify -DskipUnitTests'
            }
        }

        stage('Code analysis with sonar'){

            environment {
                scannerHome= tool 'sonarscanner'
            }

            steps{
                withSonarQubeEnv('sonar-pro'){
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                   -Dsonar.projectName=vprofile-repo \
                   -Dsonar.projectVersion=1.0 \
                   -Dsonar.sources=src/ \
                   -Dsonar.java.binaries=target/test-classes/com/visualpathit/account/controllerTest/ \
                   -Dsonar.junit.reportsPath=target/surefire-reports/ \
                   -Dsonar.jacoco.reportsPath=target/jacoco.exec'''
                }

                timeout(time: 10, unit: 'MINUTES'){
                    waitForQualityGate abortPipeline: true
                }

                }
            }

        stage('Building Docker image')
            steps{
                script{
                    dockerImage= docker.build registry + ":$BUILD_NUMBER"
                }
            }

        stage('Deploy docker image'){
            steps{
                script {
                    docker.withRegistry( '',registryCredential){}
                    dockerImage.push("$BUILD_NUMBER")
                    dockerImage.push('latest')
                }
            }
        }

        stage('Remove unused docker image'){
            steps{
                sh "docker rmi $registry:$BUILD_NUMBER"
            }
        }

        stage('deploy in kubernetes'){
            agent {label 'KOPS'}
            steps{
                sh "helm upgrade --install --force vprofile-stack helm/vprofilecharts --set appimage=${REGISTRY}:${BUILD_NUMBER} --namespace prod"
            }
        }

    }

}