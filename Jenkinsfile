pipeline {
    agent {
        any {
            image 'maven:3.3.3'
        }
    }
    stages {
        stage('build') {
            steps {
                script {
                    WHO_OUTPUT = sh (
                        script: 'who',
                        returnStdout: true
                    ).trim()


                    PWD_OUTPUT = sh (
                        script: 'pwd',
                        returnStdout: true
                    ).trim()
                }
                echo "I am : ${WHO_OUTPUT}"
                echo "At : ${PWD_OUTPUT}"

                sh 'mvn --version'
            }
        }
    }
}
