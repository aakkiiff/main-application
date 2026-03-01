pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_USERNAME = "aakkiiff"
        DOCKERHUB_REPO_NAME = "jenkinstest"
    }

    stages {


        stage('BUILD DOCKER IMAGE') {
            steps {
                sh 'docker build -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO_NAME}:${IMAGE_TAG} .'
            }
        }
        stage('DOCKER LOGIN + PUSH') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'uname')]) {
                    sh 'echo ${pass} | docker login -u ${uname} --password-stdin'
                }

                sh 'docker push ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO_NAME}:${IMAGE_TAG}'
                sh 'docker logout'
           

            }
        }
        stage('clean docker images') {
            steps {
                sh 'docker rmi ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO_NAME}:${IMAGE_TAG}'
            }
        }
        stage('CLEANING WORKSPACE') {
            steps {
                script{
                    cleanWs()
                }
            }
        }

    }

}