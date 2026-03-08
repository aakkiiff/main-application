pipeline {
    agent any

    environment {
        IMAGE_TAG           = "${BUILD_NUMBER}"
        DOCKERHUB_USERNAME  = "aakkiiff"
        DOCKERHUB_REPO_NAME = "jenkinstest"

        // sonar scanner setup
        SONAR_SCANNER_VERSION   = '7.2.0.5079'
        SONAR_SCANNER_HOME      = "${HOME}/.sonar/sonar-scanner-${SONAR_SCANNER_VERSION}-linux-x64"
        PATH                    = "${SONAR_SCANNER_HOME}/bin:${PATH}"
        SONAR_HOST_URL          = "http://13.233.124.152:9000"
        SONAR_TOKEN             = "sqp_bb769152d29694ff6a2121164e449bb021ea4648"
        SONAR_PROJECT_KEY        = "test"
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }

    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'action',  value: '$.action'],
                [key: 'base',    value: '$.pull_request.base.ref'],
                [key: 'head',    value: '$.pull_request.head.ref'],
                [key: 'success', value: '$.pull_request.merged']
            ],
            token: 'test',
            regexpFilterText: '$action $base $head $success',
            regexpFilterExpression: '^closed prod stage true$'
        )
    }

    options {
        buildDiscarder(logRotator(numToKeepStr: '3'))
    }

    triggers {
        GenericTrigger(
            genericVariables: [
                [key: 'action',  value: '$.action'],
                [key: 'base',    value: '$.pull_request.base.ref'],
                [key: 'head',    value: '$.pull_request.head.ref'],
                [key: 'success', value: '$.pull_request.merged']
            ],
            token: 'test',
            regexpFilterText: '$action $base $head $success',
            regexpFilterExpression: '^closed prod stage true$'
        )
    }

    stages {

        stage('SONARQUBE SCAN') {
            steps {
                sh '''
                sonar-scanner \
                -Dsonar.projectKey=${SONAR_PROJECT_KEY} \
                -Dsonar.sources=. \
                -Dsonar.host.url=${SONAR_HOST_URL} \
                -Dsonar.token=${SONAR_TOKEN} \
                -Dsonar.qualitygate.wait=true
                '''
            }
        }

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

                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub',
                        passwordVariable: 'pass',
                        usernameVariable: 'uname'
                    )
                ]) {
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
                build job: 'config-pipeline',
                parameters: [
                    string(name: 'IMAGE_TAG', value: "${IMAGE_TAG}")
                ]
            }
        }
        
        stage('CLEANING WORKSPACE') {
            steps {
                script {
                    cleanWs()
                }
            }
        }

    }
}