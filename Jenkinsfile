pipeline {
    agent {
        any {
            image 'maven:3.3.3'
        }
    }
    tools {
        maven 'M3'
    }
    stages {
        stage('build') {
            steps {
                script {
                    PWD_OUTPUT = sh (
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
