pipeline {
    agent any

    tools {
        jdk 'jdk17'
        nodejs 'node16'
    }

    environment {
        SCANNER_HOME=tool 'sonar-scanner'
        APP_NAME = "prod-kaiburr"
        RELEASE = "1.1.1"
        DOCKER_USER = "aasaithambi5"
        DOCKER_PASS = 'docker'
        IMAGE_NAME = "${DOCKER_USER}" + "/" + "${APP_NAME}"
        IMAGE_TAG = "${RELEASE}-${BUILD_NUMBER}"
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

        stage("Build & Push Docker Image") {
            steps {
                script {
                    docker.withRegistry('',DOCKER_PASS) {
                    docker_image = docker.build "${IMAGE_NAME}"
                    }

                    docker.withRegistry('',DOCKER_PASS) {
                    docker_image.push("${IMAGE_TAG}")
                    docker_image.push('latest')
                    }
                }
            }
        }

        stage("Trivy Scan") {
            steps {
                script {
	                sh ('docker run -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy image aasaithambi5/kaiburr:latest --no-progress --scanners vuln  --exit-code 0 --severity HIGH,CRITICAL --format table > trivyimage.txt')
               }
           }
        }

        stage ('Cleanup Artifacts') {
             steps {
                 script {
                    sh "docker rmi ${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "docker rmi ${IMAGE_NAME}:latest"
                }
            }
        }

        stage('Commit & Push Manifest') {
            steps {
                script {
                    git branch: 'master', credentialsId: 'github', url: 'https://github.com/AASAITHAMBI57/kaiburr_CD.git'
                    sh "git config user.name 'AASAITHAMBI57'"
                    sh "git config user.email 'aasaithambi57@gmail.com'"
                    sh("sed -i 's/${APP_NAME}.*/${APP_NAME}:${IMAGE_TAG}/g' kustomize-yaml/prod-env/prod-deployment.yaml")
                    
                    withCredentials([gitUsernamePassword(credentialsId: 'github', gitToolName: 'Default')]) {
                        sh 'git fetch origin'
                        sh 'git add -u'
                        sh "git commit -m 'Update deployment image to ${IMAGE_TAG}'"
                        sh 'git push origin master'
                    }
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
