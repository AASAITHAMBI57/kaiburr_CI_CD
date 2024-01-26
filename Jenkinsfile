pipeline {
    agent any

    stages {

        stage('clean workspace') {
            steps {
                cleanWs()
            }
        }
        
        stage('Checkout from Git') {
            steps {
                git 'https://github.com/AASAITHAMBI57/kaiburr_CI_CD.git'
            }
        }
    }
}
