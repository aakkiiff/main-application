pipeline {
    agent any

    environment {
        IMAGE_TAG = "${BUILD_NUMBER}"
        DOCKERHUB_USERNAME = "aakkiiff"
        DOCKERHUB_REPO_NAME = "jenkinstest"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }

    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'action', value: '$.action'],
                [key: 'base', value: '$.pull_request.base.ref'],
                [key: 'head', value: '$.pull_request.head.ref'],
                [key: 'success', value: '$.pull_request.merged']
            ],
            token: 'test',
            regexpFilterText: '$action $base $head $success',
            regexpFilterExpression: '^closed prod stage true$'
        )
    }

    stages {

        stage('BUILD DOCKER IMAGE') {
            steps {
                sh 'docker build -t ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO_NAME}:${IMAGE_TAG} .'
            }
        }

        stage('image scanning') {
            steps {
                sh 'trivy image --exit-code 1 --severity CRITICAL ${DOCKERHUB_USERNAME}/${DOCKERHUB_REPO_NAME}:${IMAGE_TAG}'
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

        stage('trigger next pipeline') {
            steps {
                build job: 'config-pipeline', parameters: [string(name: 'IMAGE_TAG', value: "${IMAGE_TAG}")]
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