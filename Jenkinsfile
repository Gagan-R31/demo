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
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    def workspaceDir = pwd()
                    sh """
                    git clone -b master https://${GITHUB_TOKEN}@github.com/Gagan-R31/demo.git
                    """
                    env.COMMIT_SHA = sh(script: "git -C ${workspaceDir}/demo rev-parse --short HEAD", returnStdout: true).trim()
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
                        // Check if the deployment already exists in the specified namespace
                        def deploymentExists = sh(
                            script: "kubectl get deployment ${DEPLOYMENT_NAME} -n ${NAMESPACE} --ignore-not-found",
                            returnStatus: true
                        ) == 0

                        if (deploymentExists) {
                            // Update the existing deployment with the new image
                            echo "Updating existing deployment with new image..."
                            sh """
                            kubectl set image deployment/${DEPLOYMENT_NAME} ${DEPLOYMENT_NAME}=${DOCKERHUB_REPO}:${COMMIT_SHA} -n ${NAMESPACE}
                            kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE}
                            kubectl get pods -n ${NAMESPACE}
                            """
                        } else {
                            // Create a new deployment if it doesn't exist
                            echo "Creating new deployment..."
                            sh """
                            kubectl create deployment ${DEPLOYMENT_NAME} --image=${DOCKERHUB_REPO}:${COMMIT_SHA} --dry-run=client -o yaml > k8s-deployment.yaml
                            kubectl apply -f k8s-deployment.yaml -n ${NAMESPACE}
                            kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE}
                            kubectl get pods -n ${NAMESPACE}
                            """
                        }
                    }
                }
            }
        }
    }
}
