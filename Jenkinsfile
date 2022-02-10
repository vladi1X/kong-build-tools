pipeline {
    agent none
    triggers {
        cron(env.BRANCH_NAME == 'master' ? '@weekly' : '')
    }
    environment {
        KONG_SOURCE = "master"
        KONG_SOURCE_LOCATION = "/tmp/kong"
        DOCKER_USERNAME = "${env.DOCKERHUB_USR}"
        DOCKER_PASSWORD = "${env.DOCKERHUB_PSW}"
        DOCKERHUB = credentials('dockerhub')
        DOCKER_CLI_EXPERIMENTAL = "enabled"
        DEBUG = 0
    }
    options {
        timeout(time: 120, unit: 'MINUTES')
        parallelsAlwaysFailFast()
        retry(2)
    }
    stages {
        stage('Test Builds') {
            when {
                beforeAgent true
                anyOf {
                    buildingTag()
                    branch 'master'
                    branch 'feat/kong-enterprise'
                    changeRequest target: 'master'
                }
            }
            parallel {
                stage('rpm'){
                    agent {
                        node {
                            label 'bionic'
                        }
                    }
                    environment {
                        AWS_ACCESS_KEY = credentials('AWS_ACCESS_KEY')
                        AWS_SECRET_ACCESS_KEY = credentials('AWS_SECRET_ACCESS_KEY')
                        PACKAGE_TYPE="rpm"
                    }
                    steps {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin || true'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'export RESTY_IMAGE_BASE=amazonlinux RESTY_IMAGE_TAG=2 && make package-kong && make test && make cleanup'
                        sh 'export RESTY_IMAGE_BASE=rhel RESTY_IMAGE_TAG=7 && make package-kong && make test && make cleanup'
                        sh 'export RESTY_IMAGE_BASE=centos RESTY_IMAGE_TAG=7 && make package-kong && make test && make cleanup'
                    }
                }
                stage('src & Alpine'){
                    agent {
                        node {
                            label 'bionic'
                        }
                    }
                    steps {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin || true'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'export RESTY_IMAGE_BASE=src RESTY_IMAGE_TAG=src PACKAGE_TYPE=src && make package-kong && make test && make cleanup'
                        sh 'export RESTY_IMAGE_BASE=alpine RESTY_IMAGE_TAG=3.10 PACKAGE_TYPE=apk && make package-kong && make test && make cleanup'
                    }
                }
                stage('deb') {
                    agent {
                        node {
                            label 'bionic'
                        }
                    }
                    environment {
                        PACKAGE_TYPE="deb"
                        PATH = "/home/ubuntu/bin/:${env.PATH}"
                    }
                    steps {
                        sh 'mkdir -p /home/ubuntu/bin/'
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin || true'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'export RESTY_IMAGE_BASE=debian RESTY_IMAGE_TAG=8 && make package-kong && make test && make cleanup'
                        sh 'export RESTY_IMAGE_BASE=debian RESTY_IMAGE_TAG=9 && make package-kong && make test && make cleanup'
                        sh 'export RESTY_IMAGE_BASE=debian RESTY_IMAGE_TAG=10 && make package-kong && make test && make cleanup'
                        sh 'export RESTY_IMAGE_BASE=debian RESTY_IMAGE_TAG=11 && make package-kong && make test && make cleanup'
                        sh 'export RESTY_IMAGE_BASE=ubuntu RESTY_IMAGE_TAG=16.04 && make package-kong && make test && make cleanup'
                        sh 'export RESTY_IMAGE_BASE=ubuntu RESTY_IMAGE_TAG=18.04 && make package-kong && make test && make cleanup'
                        sh 'export RESTY_IMAGE_BASE=ubuntu RESTY_IMAGE_TAG=20.04 && make package-kong && make test && make cleanup'
                    }
                }
            }
        }
        stage('Release') {
            agent {
                node {
                    label 'bionic'
                }
            }
            when {
                triggeredBy 'TimerTrigger'
            }
            environment {
                GITHUB_TOKEN = credentials('github_bot_access_token')
            }
            steps {
                sh 'curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.2/install.sh | bash'
                sh '. ~/.nvm/nvm.sh && nvm install lts/*'
                sh '. ~/.nvm/nvm.sh && npx semantic-release@beta'
            }
        }
    }
}
