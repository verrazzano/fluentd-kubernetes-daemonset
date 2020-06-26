// Copyright (c) 2020, Oracle Corporation and/or its affiliates.
// Licensed under the Universal Permissive License v 1.0 as shown at https://oss.oracle.com/licenses/upl.

def HEAD_COMMIT

pipeline {
    options {
      skipDefaultCheckout true
      disableConcurrentBuilds()
    }

    agent {
        docker {
            image "${RUNNER_DOCKER_IMAGE}"
            args "${RUNNER_DOCKER_ARGS}"
            registryUrl "${RUNNER_DOCKER_REGISTRY_URL}"
            registryCredentialsId 'ocir-pull-and-push-account'
        }
    }

    environment {
	FLUENTD_IMAGE_NAME = "fluentd:v1.10.4-oraclelinux-1.0"
        DOCKER_IMAGE_NAME = "fluentd-kubernetes-daemonset:v1.10.4-oraclelinux-elasticsearch7-1.0-test"
        GOPATH = "$HOME/go"
        GO_REPO_PATH = "${GOPATH}/src/github.com/verrazzano"
        DOCKER_CREDS = credentials('ocir-pull-and-push-account')
    }

    stages {
        stage('Clean workspace and checkout') {
            steps {
                checkout scm
                sh """
                    echo "${DOCKER_CREDS_PSW}" | docker login ${env.DOCKER_REPO} -u ${DOCKER_CREDS_USR} --password-stdin
                    rm -rf ${GO_REPO_PATH}/fluentd-kubernetes-daemonset
                    mkdir -p ${GO_REPO_PATH}/fluentd-kubernetes-daemonset
                    tar cf - . | (cd ${GO_REPO_PATH}/fluentd-kubernetes-daemonset/ ; tar xf -)
                """
            }
        }

        stage('Build') {
	        when {
       		    not {
           	        branch 'master'
       	            }
   	        }
            steps {
                sh """
                    cd ${GO_REPO_PATH}/fluentd-kubernetes-daemonset
		    docker image build --build-arg FLUENTD_IMG="${env.DOCKER_REPO}/${env.DOCKER_NAMESPACE}/${FLUENTD_IMAGE_NAME}" -f docker-image/v1.10/oraclelinux-elasticsearch7/Dockerfile -t ${env.DOCKER_REPO}/${env.DOCKER_NAMESPACE}/${DOCKER_IMAGE_NAME} .
                    docker image push ${env.DOCKER_REPO}/${env.DOCKER_NAMESPACE}/${DOCKER_IMAGE_NAME}
                """
            }
        }

        stage('Scan Image') {
            when {
                not {
                   branch 'master'
                }
            }
            steps {
                script {
                    HEAD_COMMIT = sh(returnStdout: true, script: "git rev-parse HEAD").trim()
                    clairScanTemp "${env.DOCKER_REPO}/${env.DOCKER_NAMESPACE}/${DOCKER_IMAGE_NAME}"
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: '**/scanning-report.json', allowEmptyArchive: true
                }
            }
        }

    }

    post {
        failure {
            mail to: "${env.BUILD_NOTIFICATION_TO_EMAIL}", from: 'noreply@oracle.com',
            subject: "Verrazzano: ${env.JOB_NAME} - Failed",
            body: "Job Failed - \"${env.JOB_NAME}\" build: ${env.BUILD_NUMBER}\n\nView the log at:\n ${env.BUILD_URL}\n\nBlue Ocean:\n${env.RUN_DISPLAY_URL}"
        }
    }
}
