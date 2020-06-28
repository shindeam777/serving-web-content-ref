pipeline {
    agent {
        any {
            image 'maven:3.3.3'
        }
    }
    stages {
        stage('build') {
            steps {
                WHO_OUTPUT = sh (
                    script: 'who',
                    returnStdout: true
                ).trim()
                echo "I am : ${WHO_OUTPUT}"

                PWD_OUTPUT = sh (
                    script: 'pwd',
                    returnStdout: true
                ).trim()
                echo "At : ${PWD_OUTPUT}"

                sh 'mvn --version'
            }
        }
    }
}
