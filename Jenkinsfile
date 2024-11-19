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
        NAMESPACE = 'jenkins-operator'
        SOURCE_BRANCH = "${env.CHANGE_BRANCH ?: env.GIT_BRANCH}"
        THREAD_ID = UUID.randomUUID().toString() // Create a unique thread ID for this build
        JENKINS_URL = "${env.BUILD_URL}" // URL to Jenkins build page
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    def workspaceDir = pwd()
                    sh """
                    git clone -b dev https://${GITHUB_TOKEN}@github.com/Gagan-R31/demo.git
                    """
                    env.COMMIT_SHA = sh(script: "git -C ${workspaceDir}/demo rev-parse --short HEAD", returnStdout: true).trim()
                }
            }
            post {
                success {
                    script {
                        def requestBody = """{
                            "text": ":white_check_mark: **Clone Repository completed successfully**\\nBranch: ${env.SOURCE_BRANCH}, Commit: ${env.COMMIT_SHA}\\n[View Build Logs](${env.JENKINS_URL})",
                            "thread_id": "${env.THREAD_ID}"
                        }"""
                        httpRequest(
                            httpMode: 'POST',
                            url: 'https://api.pumble.com/workspaces/673c21ed7f891f7d9b11d8cf/incomingWebhooks/postMessage/NrRX5lxK6ixnWnGqfQAMKRXB',
                            contentType: 'APPLICATION_JSON',
                            requestBody: requestBody
                        )
                    }
                }
                failure {
                    script {
                        def requestBody = """{
                            "text": ":x: **Clone Repository failed**\\nBranch: ${env.SOURCE_BRANCH}\\n[View Build Logs](${env.JENKINS_URL})",
                            "thread_id": "${env.THREAD_ID}"
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
        // Similarly, other stages like Build Docker Image and Deploy to K3s go here
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
            post {
                success {
                    script {
                        def requestBody = """{
                            "text": ":white_check_mark: **Docker image built successfully**\\nImage: ${DOCKERHUB_REPO}:${COMMIT_SHA}\\n[View Build Logs](${env.JENKINS_URL})",
                            "thread_id": "${env.THREAD_ID}"
                        }"""
                        httpRequest(
                            httpMode: 'POST',
                            url: 'https://api.pumble.com/workspaces/673c21ed7f891f7d9b11d8cf/incomingWebhooks/postMessage/NrRX5lxK6ixnWnGqfQAMKRXB',
                            contentType: 'APPLICATION_JSON',
                            requestBody: requestBody
                        )
                    }
                }
                failure {
                    script {
                        def requestBody = """{
                            "text": ":x: **Docker image build failed**\\nCommit: ${env.COMMIT_SHA}\\n[View Build Logs](${env.JENKINS_URL})",
                            "thread_id": "${env.THREAD_ID}"
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
    }
}
