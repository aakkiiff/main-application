pipeline {
    agent any

    stages {

        stage('BUILD DOCKER IMAGE') {
            steps {
                echo 'building docker image........'
                sh 'docker build -t aakkiiff/jenkinstest:${BUILD_NUMBER} .'
            }
        }
        stage('DOCKER LOGIN + PUSH') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', passwordVariable: 'pass', usernameVariable: 'uname')]) {
                    sh 'echo ${pass} | docker login -u ${uname} --password-stdin'
                }

                sh 'docker push aakkiiff/jenkinstest:${BUILD_NUMBER}'

                sh 'docker logout'
                echo 'pushing docker image'

            }
        }
        stage('clean docker images') {
            steps {
                sh 'docker rmi aakkiiff/jenkinstest:${BUILD_NUMBER}'
            }
        }

    }

}