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
            script {
                PROJ_NAME = "serving-web-content"
            }
            steps {
                echo "Starting Checkout and Build stage."
                //Clean up the workspace
                sh "rm -rf *"

                checkout scm

                script {
                    //Get the GIT Commit Tag
                    GIT_TAG = sh(returnStdout: true, script: "git log -1 --do-walk --pretty=format:%h").trim()
                }
                echo "GIT_TAG: ${GIT_TAG}"

                //Set the buildName to the GIT Commit Hash
                buildName "${GIT_TAG}"

                echo "The build name is set to: ${env.BUILD_DISPLAY_NAME}"

                script {
                    PWD_OUTPUT = sh(
                            script: 'pwd',
                            returnStdout: true
                    ).trim()
                }
                echo "Current path : ${PWD_OUTPUT}"

                sh 'mvn clean install'
            }
            post {
                success {
                    echo "Checkout and build completed successfully!"

                    //bundle and stash the project jar files
                    sh "tar cf ${PROJ_NAME}-bin.tar `find . -name *.jar | grep -v \"\\.m2\"`"
                    stash includes: "${PROJ_NAME}-bin.tar", name: "${PROJ_NAME}-bin"

                    //Stash the code for later use
                    sh "tar cf ${PROJ_NAME}-code.tar `find . -name \"*\" | grep -v -e \"\\..ar\" -e \"\\.m2\" -e \"\\.git\" `"
                    stash includes: "${PROJ_NAME}-code.tar", name: "${PROJ_NAME}-code"

                    //Stash the docker file
                    stash includes: "Dockerfile", name: "${PROJ_NAME}-Dockerfile"
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
                    DOCKER_IMAGE_TAG = "${ORG_NAME}/${PROJ_NAME}:$GIT_TAG"
                }

                //Unstash the files and extract
                unstash "${PROJ_NAME}-bin"
                sh "tar xf ${PROJ_NAME}-bin.tar"

                unstash "${PROJ_NAME}-Dockerfile"

                //publish code goes here
            }
            post {
                success {
                    echo "Docker image[$DOCKER_IMAGE_TAG] created and published."
                }
                failure {
                    echo "Docker image[$DOCKER_IMAGE_TAG] creation or publishing failed."
                }
                cleanup {
                    //Clean-up the workspace
                    cleanUpWorkSpace("${STAGE_NAME}")
                }
            }
        }
    }
}
