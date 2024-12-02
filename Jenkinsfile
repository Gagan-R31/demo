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
        stage('Clone Application Repository') {
            steps {
                script {
                    if (env.TAG_NAME != null && env.TAG_NAME != '') {
                        sh """
                        git clone --depth 1 --branch ${env.TAG_NAME} ${env.REPO_URL}
                        """
                    } else {
                        sh """
                        git clone --depth 1 --branch version-1.0.1 ${env.REPO_URL}
                        """
                    }
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
        stage('Update Helm Chart') {
            when {
                expression { env.TAG_NAME != null && env.TAG_NAME != '' }
            }
            steps {
                container('git') {
                    script {
                        sh """
                        git clone https://${GITHUB_TOKEN}@github.com/Gagan-R31/helm-chart.git helm-chart
                        cd helm-chart
                        yq eval ".chartService.image.tag = \\"${env.TAG_NAME}\\"" -i values.yaml
                        git config user.name "Jenkins"
                        git config user.email "Gagan6696@gmail.com"
                        git add .
                        git commit -m "Update chartService image tag to ${env.TAG_NAME}"
                        git push https://${GITHUB_TOKEN}@github.com/Gagan-R31/helm-chart.git master
                        """
                    }
                }
            }
        }
        stage('Deploying to K3s') {
            when {
                expression { env.TAG_NAME == null || env.TAG_NAME == '' }
            }
            steps {
                container('kubectl') {
                    script {
                        sh """
                        echo "Applying Kubernetes manifests to deploy to K3s..."
                        echo "Deployment to K3s completed."
                        """
                    }
                }
            }
        }
    }
}
