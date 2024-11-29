pipeline {
    agent {
        kubernetes {
            label 'jenkins-k8s-agent'
            yaml """
            apiVersion: v1
            kind: Pod
            metadata:
              labels:pipeline {
    agent {
        kubernetes {
            label 'jenkins-k8s-agent'
            yaml """
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                jenkins-agent: jenkins-agent
            spec:
              nodeSelector:
                kubernetes.io/hostname: node-05-infra
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
                - name: docker-secret
                  mountPath: /kaniko/.docker
                - name: workspace-volume
                  mountPath: /workspace
              - name: kubectl
                image: boxboat/kubectl
                command:
                - cat
                tty: true
              volumes:
              - name: docker-secret
                secret:
                  secretName: acr-secret
                  items:
                  - key: .dockerconfigjson
                    path: config.json
              - name: workspace-volume
                emptyDir: {}
            """
        }
    }
    environment {
        GITHUB_TOKEN = credentials('Anand')
        DOCKERHUB_REPO = 'gaganr31/demo'
        SOURCE_BRANCH = "${env.CHANGE_BRANCH ?: env.GIT_BRANCH}"
        REPO_URL = 'https://github.com/Gagan-R31/demo.git'
        DEPLOYMENT_NAME = 'demo'
        CONTAINER_NAME = 'demo'
        NAMESPACE = 'default'
        THREAD_ID = UUID.randomUUID().toString()
        JENKINS_URL = "${env.BUILD_URL}"
        PUMBLE_URL = credentials('pumble-webhook')
    }
    stages {
        stage('Notify Pipeline Start') {
            steps {
                script {
                    sh """
                    git clone -b dev https://github.com/Gagan-R31/demo.git
                    """
                    env.COMMIT_SHA = sh(script: "git -C demo rev-parse --short HEAD", returnStdout: true).trim()
                    def requestBody = """{
                        "text": ":hourglass_flowing_sand: **Pipeline started**\\nBranch: ${SOURCE_BRANCH}, Deployment: ${DEPLOYMENT_NAME}, Commit: ${env.COMMIT_SHA}\\n[View Build Logs](${JENKINS_URL})",
                        "thread_id": "${THREAD_ID}"
                    }"""
                    httpRequest(
                        httpMode: 'POST',
                        url: "${PUMBLE_URL}",
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
                    ? ":white_check_mark: **Pipeline succeeded**\\nBranch: ${SOURCE_BRANCH},Deployment: ${DEPLOYMENT_NAME}, Commit: ${env.COMMIT_SHA}\\n[View Build Logs](${JENKINS_URL})" 
                    : ":x: **Pipeline failed**\\nBranch: ${SOURCE_BRANCH}, Commit: ${env.COMMIT_SHA}\\n[View Build Logs](${JENKINS_URL})"
                def requestBody = """{
                    "text": "${notificationText}",
                    "thread_id": "${THREAD_ID}"
                }"""
                httpRequest(
                    httpMode: 'POST',
                    url: "${PUMBLE_URL}",
                    contentType: 'APPLICATION_JSON',
                    requestBody: requestBody
                )
            }
        }
    }
}
                jenkins-agent: jenkins-agent
            spec:
              nodeSelector:
                kubernetes.io/hostname: node-05-infra
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
                - name: docker-secret
                  mountPath: /kaniko/.docker
                - name: workspace-volume
                  mountPath: /workspace
              - name: kubectl
                image: boxboat/kubectl
                command:
                - cat
                tty: true
              volumes:
              - name: docker-secret
                secret:
                  secretName: acr-secret
                  items:
                  - key: .dockerconfigjson
                    path: config.json
              - name: workspace-volume
                emptyDir: {}
            """
        }
    }
    environment {
        IMAGE_TAG = "v${BUILD_NUMBER}"
        HARBOR_REPO = "harbor.zapto.org/browny/my-app"
        HELM_CHART_DIR = "helm"
        KUBE_NAMESPACE = "browny"
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    sh """
                    echo "Cloning repository..."
                    git clone -b dev https://github.com/Gagan-R31/demo.git 
                    """
                }
            }
        }
        stage('Build and Push Image') {
            steps {
                container('kaniko') {
                    sh """
                    echo "Building and pushing the Docker image to Harbor..."
                    /kaniko/executor \
                    --dockerfile=./Dockerfile \
                    --context=. \
                    --destination=${HARBOR_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Helm Chart') {
            steps {
                container('yq') {
                    script {
                        sh """
                        echo "Cloning Helm chart repository..."
                        git clone https://github.com/Gagan-R31/demo.git helm-repo
                        cd helm-repo/${HELM_CHART_DIR}

                        echo "Updating Helm chart with new image tag..."
                        yq e '.image.tag = "${IMAGE_TAG}"' -i values.yaml

                        git config user.name "CI Bot"
                        git config user.email "gagankumar4882@gmail.com"
                        git commit -am "Update image tag to ${IMAGE_TAG}"
                        git push origin dev
                        """
                    }
                }
            }
        }
        stage('Deploy Using Helm') {
            steps {
                container('kubectl') {
                    sh """
                    echo "Deploying application with Helm..."
                    helm upgrade --install my-app ${HELM_CHART_DIR} \
                        --namespace ${KUBE_NAMESPACE} \
                        --set image.repository=${HARBOR_REPO} \
                        --set image.tag=${IMAGE_TAG} \
                        --atomic --wait
                    """
                }
            }
        }
    }
    post {
        success {
            echo "Pipeline executed successfully! Image: ${HARBOR_REPO}:${IMAGE_TAG}"
        }
        failure {
            echo "Pipeline failed! Check logs for details."
        }
    }
}
