pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {
        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }

        stage('Checkout from Git') {
            steps {
                git 'https://github.com/AASAITHAMBI57/kaiburr_ci_cd.git'
            }
        }

        stage("Sonarqube Analysis "){
            steps{
                withSonarQubeEnv('sonarqube') {
                    sh ''' $SCANNER_HOME/bin/sonar-scanner -Dsonar.projectName=Kaiburr \
                    -Dsonar.projectKey=. '''
                }
            }
        }

        stage("quality gate"){
           steps {
                script {
                    waitForQualityGate abortPipeline: false, credentialsId: 'sonarqube' 
                }
            } 
        }

        stage('Install Dependencies') {
            steps {
                sh "npm install"
            }
        }

        stage('TRIVY FS SCAN') {
            steps {
                sh "trivy fs . > trivyfs.txt"
            }
        }

        stage("Docker Build & Push"){
            steps{
                script{
                   withDockerRegistry(credentialsId: 'docker', toolName: 'docker'){ 
                       sh "docker build -t kaiburr ."
                       sh "docker tag kaiburr aasaithambi5/kaiburr:latest "
                       sh "docker push aasaithambi5/kaiburr:latest "
                       sh "docker push aasaithambi5/kaiburr:${BUILD_NUMBER} "
                    }
                }
            }
        }

        stage("TRIVY IMAGE SCAN"){
            steps{
                sh "trivy image aasaithambi5/kaiburr:latest > trivy.txt" 
            }
        }

        stage ('Cleanup Artifacts') {
            steps {
                script {
                    sh "docker rmi aasaithambi5/kaiburr:latest "
                    sh "docker rmi aasaithambi5/kaiburr:${BUILD_NUMBER} "
                }
            }
        }
    }
    post {
     always {
        emailext attachLog: true,
            subject: "'${currentBuild.result}'",
            body: "Project: ${env.JOB_NAME}<br/>" +
            "Build Number: ${env.BUILD_NUMBER}<br/>" +
            "URL: ${env.BUILD_URL}<br/>",
            to: 'aasaithambi5793@gmail.com', 
            attachmentsPattern: 'trivyfs.txt,trivyimage.txt'
        }
    }
}
