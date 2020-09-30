pipeline {
    agent none
    triggers {
        cron(env.BRANCH_NAME == 'master' ? '@weekly' : '')
    }
    environment {
        KONG_SOURCE = "next"
        KONG_SOURCE_LOCATION = "/tmp/kong"
        DOCKER_USERNAME = "${env.DOCKERHUB_USR}"
        DOCKER_PASSWORD = "${env.DOCKERHUB_PSW}"
        DOCKERHUB = credentials('dockerhub')
        DOCKER_CLI_EXPERIMENTAL = "enabled"
    }
    options {
        timeout(time: 120, unit: 'MINUTES')
        parallelsAlwaysFailFast()
    }
    stages {
        stage('Centos Builds'){
            agent {
                node {
                    label 'bionic'
                }
            }
            environment {
                DEBUG = 0
                PACKAGE_TYPE = "rpm"
                RESTY_IMAGE_BASE = "centos"
                PATH = "/home/ubuntu/bin/:${env.PATH}"
                PRIVATE_KEY_FILE = credentials('kong.private.gpg-key.asc')
                PRIVATE_KEY_PASSPHRASE = credentials('kong.private.gpg-key.asc.password')
            }
            steps {
                sh 'ls .'
                sh 'cp $PRIVATE_KEY_FILE ./kong.private.gpg-key.asc'
                sh 'ls .'
                sh 'mkdir -p /home/ubuntu/bin/'
                sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                sh 'export RESTY_IMAGE_TAG=8 && make package-kong'
                archiveArtifacts artifacts: 'output/*', fingerprint: true
            }
        }
    }
}
