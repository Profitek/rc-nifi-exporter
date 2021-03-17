pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    parameters {
        string(defaultValue: '', description: 'new version to release, required to publish docker image', name: 'NEW_VERSION')
        string(defaultValue: '', description: 'use existent version, use it if you want to mark an existing release as latest docker image, this will make it default (define only number e.g 0.28.0', name: 'EXISTENT_TAG')
    }

    stages {
        stage('Checkout') {
            when {
                expression {
                    !params.EXISTENT_TAG?.trim()
                }
            }
            steps {
                checkout scm
                sh("printenv")
            }
        }

        stage('Build') {
            when {
                expression {
                    !params.EXISTENT_TAG?.trim()
                }
            }
            steps {
                loadBuildInfo()
                sh "docker build -t ${APP_NAME} ."
            }
        }

        stage('Release') {
            when {
                expression {
                    env.BRANCH_NAME == 'master' &&
                            !params.EXISTENT_TAG?.trim() &&
                            params.NEW_VERSION?.trim()
                }
            }
            steps {
                sh "git tag -a ${APP_VERSION} -m 'Release ${APP_VERSION} from Jenkinsfile'"
                sh "git push origin --tags"
            }
        }

        stage('Restore TAG') {
            when {
                expression {
                    env.BRANCH_NAME == 'master' &&
                            params.EXISTENT_TAG?.trim()
                }
            }
            steps {
                loadBuildInfo()
                sh '$(aws ecr get-login --no-include-email --region sa-east-1)'
                sh "docker tag ${IMAGE_NAME} 700489516080.dkr.ecr.sa-east-1.amazonaws.com/${APP_NAME}:latest"
                sh "docker push 700489516080.dkr.ecr.sa-east-1.amazonaws.com/${APP_NAME}:latest"
            }
        }

        stage('Publishing Docker Image') {
            when {
                expression {
                    env.BRANCH_NAME == 'master' &&
                            !params.EXISTENT_TAG?.trim() &&
                            params.NEW_VERSION?.trim()
                }
            }
            steps {
                script {
                    sh '$(aws ecr get-login --no-include-email --region sa-east-1)'
                    sh "docker tag ${APP_NAME} 700489516080.dkr.ecr.sa-east-1.amazonaws.com/${APP_NAME}:${APP_VERSION}"
                    sh "docker push 700489516080.dkr.ecr.sa-east-1.amazonaws.com/${APP_NAME}:${APP_VERSION}"
                    //sh "docker tag ${APP_NAME} 700489516080.dkr.ecr.sa-east-1.amazonaws.com/${APP_NAME}:latest"
                    //sh "docker push 700489516080.dkr.ecr.sa-east-1.amazonaws.com/${APP_NAME}:latest"
                }
            }
        }
    }
}

def loadBuildInfo() {

    env.APP_NAME = 'rc-nifi-exporter'
    env.REGISTRY = "700489516080.dkr.ecr.sa-east-1.amazonaws.com"
    if (params.EXISTENT_TAG?.trim()) {
        env.APP_VERSION = params.EXISTENT_TAG?.trim()
    } else if (params.NEW_VERSION?.trim()) {
        env.APP_VERSION = params.NEW_VERSION?.trim()
    } else {
        env.APP_VERSION = 'snapshot'
    }
    env.IMAGE_NAME = "${REGISTRY}/${APP_NAME}:${APP_VERSION}"

    echo "IMAGE_NAME: ${env.IMAGE_NAME}"
    echo "APP_NAME: ${env.APP_NAME}"
    echo "APP_VERSION: ${env.APP_VERSION}"
    echo "IMAGE_URL: ${env.REGISTRY}/${env.APP_NAME}:${env.APP_VERSION}"

}