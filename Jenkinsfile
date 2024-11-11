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
        DEPLOYMENT_NAME = 'browny'
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    def workspaceDir = pwd()
                    sh """
                    git clone ${REPO_URL} ${workspaceDir}/demo
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
                        sh """
                        kubectl create deployment ${DEPLOYMENT_NAME} --image=${DOCKERHUB_REPO}:${COMMIT_SHA} --dry-run=client -o yaml > k8s-deployment.yaml
                        kubectl apply -f k8s-deployment.yaml
                        kubectl rollout status deployment/${DEPLOYMENT_NAME}
                        kubectl get pods
                        """
                    }
                }
            }
        }
    }
}
