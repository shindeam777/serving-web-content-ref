pipeline {
    agent any
    tools {
        maven 'M3'
    }
    stages {
        stage('build') {
//            agent {
//                docker { image 'maven:3-alpine' }
//            }
            steps {
                script {
                    PWD_OUTPUT = sh(
                            script: 'pwd',
                            returnStdout: true
                    ).trim()
                }
                echo "Current path : ${PWD_OUTPUT}"

                sh 'mvn clean install'
            }
        }
    }
}
