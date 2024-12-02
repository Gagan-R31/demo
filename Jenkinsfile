pipeline {
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
              - name: git
                image: alpine/git
                command:
                - cat
                tty: true
              - name: yq
                image: mikefarah/yq:latest
                command:
                - cat
                tty: true
              volumes:
              - name: docker-secret
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
        DOCKERHUB_REPO = 'gaganr31/chat-service'
        GITHUB_TOKEN = credentials('github-token-gagan')
        REPO_URL = 'https://github.com/Gagan-R31/demo.git'
        HELM_CHART_REPO = 'https://github.com/Gagan-R31/helm-chart.git'
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    def workspaceDir = pwd()
                    sh """
                    git clone --depth 1 --branch ${env.TAG_NAME} ${env.REPO_URL}
                    """
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
                                         --destination=${DOCKERHUB_REPO}:${env.TAG_NAME}
                        """
                    }
                }
            }
        }
        stage('Clone Repository') {
            steps {
                script {
                    def workspaceDir = pwd()
                    sh """
                    git clone ${HELM_CHART_REPO}
                    """
                }
            }
        }
        stage('Update Helm Chart') {
            steps {
                container('yq') {
                    script {
                        sh """
                        cd helm-chart
                        yq eval ".chartService.image.tag = \\"${env.TAG_NAME}\\"" -i helm-charts/values.yaml
                        """
                    }
                }
            }
        }
        stage('Commit and Push Helm Chart Changes') {
            steps {
                container('git') {
                    script {
                        sh """
                        cd helm-chart
                        git config user.name "Jenkins"
                        git config user.email "Gagan6696@gmail.com"
                        git add .
                        git commit -m "Update chartService image tag to ${env.TAG_NAME}"
                        git push origin master
                        """
                    }
                }
            }
        }
    }
}
