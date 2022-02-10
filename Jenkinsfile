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
    }
    stages {
        stage('build and test') {
            matrix {
                when {
                    beforeAgent true
                    anyOf {
                        buildingTag()
                        branch 'master'
                        branch 'feat/kong-enterprise'
                        changeRequest target: 'master'
                    }
                }
                agent {
                    node {
                        label 'bionic'
                    }
                }
                axes {
                    axis {
                        name 'PACKAGE_TYPE'
                        values 'apk', 'rpm', 'deb'
                    }
                    axis {
                        name 'KONG_GITHUB_REPOSITORY'
                        values 'git@github.com:Kong/kong.git', 'git@github.com:Kong/kong-ee.git'
                    }
                }
                stages {
                    stage('setup') {
                        steps {
                            sh 'echo "$DOCKER_PASSWORD" | docker login -u "$DOCKER_USERNAME" --password-stdin || true'
                            sh 'git clone --recursive --single-branch --branch ${KONG_SOURCE} ${KONG_GITHUB_REPOSITORY} ${KONG_SOURCE_LOCATION}'
                        }
                    }
                    stage('build') {
                        steps {
                            sh 'make package-kong'
                        }
                    }
                    stage('test') {
                        steps {
                            sh 'make test'
                        }
                    }
                }
            }
        }
    }
}
