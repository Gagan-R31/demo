pipeline {
    agent {
        kubernetes {
            label 'k8s-agent'
            yaml """
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                some-label: some-label-value
            spec:
              serviceAccountName: jenkins
              containers:
              - name: jnlp
                image: jenkins/inbound-agent
                args: ['\$(JENKINS_SECRET)', '\$(JENKINS_NAME)']
              - name: kaniko
                image: gaganr31/kaniko-go
                command:
                - /busybox/sh
                tty: true
                volumeMounts:
                - name: kaniko-secret
                  mountPath: /kaniko/.docker
                - name: workspace-volume
                  mountPath: /workspace
              - name: kubectl
                image: boxboat/kubectl
                command:
                - cat
                tty: true
              volumes:
              - name: kaniko-secret
                secret:
                  secretName: kaniko-secret
                  items:
                  - key: .dockerconfigjson
                    path: config.json
              - name: workspace-volume
                emptyDir: {}
            """
        }
    }
    environment {
        GITHUB_TOKEN = credentials('github-token')
        DOCKERHUB_REPO = 'gaganr31/argu'
        REPO_URL = 'https://github.com/Gagan-R31/demo.git'
        DEPLOYMENT_NAME = 'argu'
        NAMESPACE = 'test'
        CONTAINER_NAME = 'argu'
        SOURCE_BRANCH = "${env.CHANGE_BRANCH ?: env.GIT_BRANCH}"
        THREAD_ID = UUID.randomUUID().toString()
        JENKINS_URL = "${env.BUILD_URL}"
    }
    stages {
        stage('Notify Pipeline Start') {
            steps {
                script {
                    sh """
                    git clone -b dev https://${GITHUB_TOKEN}@github.com/Gagan-R31/demo.git
                    """
                    env.COMMIT_SHA = sh(script: "git -C demo rev-parse --short HEAD", returnStdout: true).trim()
                    def requestBody = """{
                        "text": ":hourglass_flowing_sand: **Pipeline started**\\nBranch: ${SOURCE_BRANCH}, Commit: ${env.COMMIT_SHA}\\n[View Build Logs](${JENKINS_URL})",
                        "thread_id": "${THREAD_ID}"
                    }"""
                    httpRequest(
                        httpMode: 'POST',
                        url: 'https://api.pumble.com/workspaces/673c21ed7f891f7d9b11d8cf/incomingWebhooks/postMessage/NrRX5lxK6ixnWnGqfQAMKRXB',
                        contentType: 'APPLICATION_JSON',
                        requestBody: requestBody
                    )
                }
            }
        }
        stage('Build Docker Image with Kaniko') {
            steps {
                container('kaniko') {
                    script {
                        sh """
                        cd demo
                        /kaniko/executor --dockerfile=./Dockerfile \
                                         --context=. \
                                         --destination=${DOCKERHUB_REPO}:${COMMIT_SHA}
                        """
                    }
                }
            }
        }
        stage('Deploy to K3s') {
            steps {
                container('kubectl') {
                    script {
                        def previousImage = sh(
                            script: """
                            kubectl get deployment ${DEPLOYMENT_NAME} -n ${NAMESPACE} \
                            -o jsonpath='{.spec.template.spec.containers[?(@.name=="${CONTAINER_NAME}")].image}' || echo ""
                            """,
                            returnStdout: true
                        ).trim()

                        echo "Previous image: ${previousImage}"

                        try {
                            // Deploy new image
                            sh """
                            kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${DOCKERHUB_REPO}:${COMMIT_SHA} -n ${NAMESPACE}
                            kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE} --timeout=60s
                            """

                            // Ensure pods are running
                            def podStatus = sh(
                                script: """
                                kubectl get pods -n ${NAMESPACE} -l app=${DEPLOYMENT_NAME} \
                                -o jsonpath='{.items[0].status.phase}'
                                """,
                                returnStdout: true
                            ).trim()

                            if (podStatus != 'Running') {
                                error "Pod status is ${podStatus}. Triggering rollback."
                            }

                        } catch (Exception e) {
                            if (previousImage) {
                                echo "Rolling back to previous image: ${previousImage}"
                                sh """
                                kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${previousImage} -n ${NAMESPACE}
                                kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE}
                                """
                            } else {
                                error "No previous image found to roll back to."
                            }
                        }
                    }
                }
            }
        }
    }
    post {
        always {
            script {
                def pipelineStatus = currentBuild.result ?: 'SUCCESS'
                def notificationText = pipelineStatus == 'SUCCESS' 
                    ? ":white_check_mark: **Pipeline succeeded**\\nBranch: ${SOURCE_BRANCH}, Commit: ${env.COMMIT_SHA}\\n[View Build Logs](${JENKINS_URL})" 
                    : ":x: **Pipeline failed**\\nBranch: ${SOURCE_BRANCH}, Commit: ${env.COMMIT_SHA}\\n[View Build Logs](${JENKINS_URL})"
                def requestBody = """{
                    "text": "${notificationText}",
                    "thread_id": "${THREAD_ID}"
                }"""
                httpRequest(
                    httpMode: 'POST',
                    url: 'https://api.pumble.com/workspaces/673c21ed7f891f7d9b11d8cf/incomingWebhooks/postMessage/NrRX5lxK6ixnWnGqfQAMKRXB',
                    contentType: 'APPLICATION_JSON',
                    requestBody: requestBody
                )
            }
        }
    }
}
