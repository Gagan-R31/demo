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
        DOCKERHUB_REPO = 'gaganr31/helm-chart'
        REPO_URL = 'https://github.com/Gagan-R31/demo.git'
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
                                         --destination=${DOCKERHUB_REPO}:${env.BRANCH_NAME}
                        """
                    }
                }
            }
        }
        stage('Check Tag or Branch') {  // Removed nested 'stages' here
            steps {
                script {
                    if (env.TAG_NAME) {
                        echo "Pipeline triggered by tag: ${env.TAG_NAME}"
                    } else {
                        echo "Pipeline triggered by branch: ${env.GIT_BRANCH}"
                    }
                }
            }
        }
    }
}
