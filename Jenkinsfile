#!/usr/bin/env groovy

node('gremlin') {

    def image = docker.image('bayesian/data-model-importer')

    stage('Checkout') {
        checkout scm
    }

    stage('Tests') {
        dockerCleanup()
        timeout(30) {
            sh './runtests.sh'
        }
    }

    stage('Build') {
        docker.build(image.id, '-f Dockerfile.data-model --pull --no-cache .')
    }

    if (env.BRANCH_NAME == 'master') {
        stage('Push Images') {
            def commitId = sh(returnStdout: true, script: 'git rev-parse --short HEAD').trim()
            docker.withRegistry('https://docker-registry.usersys.redhat.com/') {
                image.push('latest')
                image.push(commitId)
            }
            docker.withRegistry('https://registry.devshift.net/') {
                image.push('latest')
                image.push(commitId)
            }
        }
    }
}

if (env.BRANCH_NAME == 'master') {
    node('oc') {
        stage('Deploy - dev') {
            sh 'oc --context=dev deploy bayesian-data-importer --latest'
        }

        stage('Deploy - rh-idev') {
            sh 'oc --context=rh-idev deploy bayesian-data-importer --latest'
        }
    }
}
