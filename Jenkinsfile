pipeline {
    agent none
    environment {
        UPDATE_CACHE = "true"
        DOCKER_CREDENTIALS = credentials('dockerhub')
        DOCKER_USERNAME = "${env.DOCKER_CREDENTIALS_USR}"
        DOCKER_PASSWORD = "${env.DOCKER_CREDENTIALS_PSW}"
        KONG_PACKAGE_NAME = 'kong'
        REPOSITORY_OS_NAME = "${env.BRANCH_NAME}"
    }
    stages {
        stage('Nightly Releases') {
            parallel {
                stage('Centos Releases') {
                    agent {
                        node {
                            label 'docker-compose'
                        }
                    }
                    environment {
                        PACKAGE_TYPE = 'rpm'
                        RESTY_IMAGE_BASE = 'centos'
                        KONG_SOURCE_LOCATION = "${env.WORKSPACE}"
                        KONG_BUILD_TOOLS_LOCATION = "${env.WORKSPACE}/../kong-build-tools"
                        REDHAT_CREDENTIALS = credentials('redhat')
                        REDHAT_USERNAME = "${env.REDHAT_USR}"
                        REDHAT_PASSWORD = "${env.REDHAT_PSW}"
                        BINTRAY_USR = 'kong-inc_travis-ci@kong'
                        BINTRAY_KEY = credentials('bintray_travis_key')
                        PRIVATE_KEY_FILE = credentials('kong.private.gpg-key.asc')
                        PRIVATE_KEY_PASSPHRASE = credentials('kong.private.gpg-key.asc.password')
                    }
                    steps {
                        sh 'make setup-kong-build-tools'
                        sh 'mkdir -p $HOME/bin'
                        sh 'sudo ln -s $HOME/bin/kubectl /usr/local/bin/kubectl'
                        sh 'sudo ln -s $HOME/bin/kind /usr/local/bin/kind'
                        dir('../kong-build-tools'){ sh 'make setup-ci' }
                        sh 'cp $PRIVATE_KEY_FILE ../kong-build-tools/kong.private.gpg-key.asc'
                        sh 'REPOSITORY_NAME=`basename ${GIT_URL%.*}`-nightly KONG_VERSION=`date +%Y-%m-%d` RESTY_IMAGE_TAG=7 make nightly-release'
                        sh 'REPOSITORY_NAME=`basename ${GIT_URL%.*}`-nightly KONG_VERSION=`date +%Y-%m-%d` RESTY_IMAGE_TAG=8 make nightly-release'
                    }
                }
            }
        }
    }
}
