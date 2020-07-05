#!/usr/bin/env groovy
pipeline {
    agent any
//    tools {
//        maven 'M3'
//    }
    stages {
        stage('build') {

            agent {
                docker { image 'maven:3-alpine' }
            }
            steps {
                echo "Starting Checkout and Build stage."
                checkout scm


                echo "The build name is set to: ${env.BUILD_DISPLAY_NAME}"

                script {
                    PWD_OUTPUT = sh(
                            script: 'pwd',
                            returnStdout: true
                    ).trim()
                }
                echo "Current path : ${PWD_OUTPUT}"

                sh 'mvn clean install -DskipTests'
            }
            post {
                success {
                    echo "Checkout and build completed successfully!"

                    //bundle and stash the project jar files
                    sh "tar cf serving-web-content-bin.tar `find . -name *.jar | grep -v \"\\.m2\"`"
                    stash includes: "serving-web-content-bin.tar", name: "serving-web-content-bin"

                    //Stash the code for later use
                    sh "tar cf serving-web-content-code.tar `find . -name \"*\" | grep -v -e \"\\..ar\" -e \"\\.m2\" -e \"\\.git\" `"
                    stash includes: "serving-web-content-code.tar", name: "serving-web-content-code"

                    //Stash the docker file
                    stash includes: "Dockerfile", name: "serving-web-content-Dockerfile"
                }
                failure {
                    echo "Checkout and build failed!!!"
                }
//                cleanup {
//                    //Clean-up the workspace
//                    cleanUpWorkSpace("${STAGE_NAME}")
//                }
            }
        }
        stage('Dockerization') {
            agent any
            steps {
                echo "Starting Dockerization stage"

                script {
                    //Docker Variables
                    DOCKER_IMAGE_TAG = "jensonjoseph/spring-boot-docker:0.0.2-SNAPSHOT"
                }

                //Unstash the files and extract
                unstash "serving-web-content-bin"
                sh "tar xf serving-web-content-bin.tar"

                unstash "serving-web-content-Dockerfile"

                script {
                    echo "Creating and publishing docker image"
                    docker.withRegistry('', "JENSON_DOCKER_HUB") {
                        def dockerImage = docker.build("${DOCKER_IMAGE_TAG}")
                        dockerImage.push()
                    }
                }
            }
            post {
                success {
                    echo "Docker image[$DOCKER_IMAGE_TAG] created and published."
                }
                failure {
                    echo "Docker image[$DOCKER_IMAGE_TAG] creation or publishing failed."
                }
//                cleanup {
//                    //Clean-up the workspace
//                    cleanUpWorkSpace("${STAGE_NAME}")
//                }
            }
        }
    }
}
