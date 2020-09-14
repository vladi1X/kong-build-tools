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
        ARCHITECTURE = "armv7hf"
    }
    stages {
        stage('Build Base') {
            when {
                beforeAgent true
                anyOf {
                    buildingTag()
                    branch 'chore/multi-arch'
                    changeRequest target: 'master'
                }
            }
            parallel {
                stage('Alpine'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    steps {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin || true'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'export RESTY_IMAGE_BASE=alpine RESTY_IMAGE_TAG=latest PACKAGE_TYPE=apk && make build-base || true'
                    }
                }
                stage('Debian'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        DEBUG = 0
                        PACKAGE_TYPE = "deb"
                        RESTY_IMAGE_BASE = "debian"
                        PATH = "/home/ubuntu/bin/:${env.PATH}"
                    }
                    steps {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin || true'
                        sh 'mkdir -p /home/ubuntu/bin/'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'export RESTY_IMAGE_TAG=stretch && make build-base || true'
                        sh 'export RESTY_IMAGE_TAG=buster && make build-base || true'
                        sh 'export RESTY_IMAGE_TAG=bullseye && make build-base || true'
                    }
                }
                stage('Ubuntu'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    options {
                        retry(2)
                    }
                    environment {
                        DEBUG = 0
                        PACKAGE_TYPE = "deb"
                        RESTY_IMAGE_BASE = "ubuntu"
                        PATH = "/home/ubuntu/bin/:${env.PATH}"
                    }
                    steps {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin || true'
                        sh 'mkdir -p /home/ubuntu/bin/'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'RESTY_IMAGE_TAG=bionic && make build-base || true'
                        sh 'RESTY_IMAGE_TAG=xenial && make build-base || true'
                        sh 'RESTY_IMAGE_TAG=focal && make build-base || true'
                    }
                }
            }
        }
        stage('Build Openresty') {
            when {
                beforeAgent true
                anyOf {
                    buildingTag()
                    branch 'chore/multi-arch'
                    changeRequest target: 'master'
                }
            }
            parallel {
                stage('Alpine'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    steps {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin || true'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'export RESTY_IMAGE_BASE=alpine RESTY_IMAGE_TAG=latest PACKAGE_TYPE=apk && make build-openresty || true'
                    }
                }
                stage('Debian'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        DEBUG = 0
                        PACKAGE_TYPE = "deb"
                        RESTY_IMAGE_BASE = "debian"
                        PATH = "/home/ubuntu/bin/:${env.PATH}"
                    }
                    steps {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin || true'
                        sh 'mkdir -p /home/ubuntu/bin/'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'export RESTY_IMAGE_TAG=stretch && make build-openresty || true'
                        sh 'export RESTY_IMAGE_TAG=buster && make build-openresty || true'
                        sh 'export RESTY_IMAGE_TAG=bullseye && make build-openresty || true'
                    }
                }
                stage('Ubuntu'){
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    options {
                        retry(2)
                    }
                    environment {
                        DEBUG = 0
                        PACKAGE_TYPE = "deb"
                        RESTY_IMAGE_BASE = "ubuntu"
                        PATH = "/home/ubuntu/bin/:${env.PATH}"
                    }
                    steps {
                        sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin || true'
                        sh 'mkdir -p /home/ubuntu/bin/'
                        sh 'git clone --single-branch --branch ${KONG_SOURCE} https://github.com/Kong/kong.git ${KONG_SOURCE_LOCATION}'
                        sh 'RESTY_IMAGE_TAG=bionic && make build-openresty || true'
                        sh 'RESTY_IMAGE_TAG=xenial && make build-openresty || true'
                        sh 'RESTY_IMAGE_TAG=focal && make build-openresty || true'
                    }
                }
            }
        }
    }
}
