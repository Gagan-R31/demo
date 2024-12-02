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
        DOCKERHUB_REPO = 'gaganr31/argu'
        REPO_URL = 'https://github.com/Gagan-R31/demo.git'
        DEPLOYMENT_NAME = 'argu'
        NAMESPACE = 'jenkins-operator'
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    def workspaceDir = pwd()
                    sh """
                    git clone https://${GITHUB_TOKEN}@github.com/Gagan-R31/demo.git
                    """
                    // Extract the branch name explicitly
                    env.BRANCH_NAME = sh(
                        script: "git -C ${workspaceDir}/demo symbolic-ref --short HEAD || echo \$GIT_BRANCH",
                        returnStdout: true
                    ).trim()
                    env.COMMIT_SHA = sh(
                        script: "git -C ${workspaceDir}/demo rev-parse --short HEAD",
                        returnStdout: true
                    ).trim()
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
                                         --destination=${DOCKERHUB_REPO}:${BRANCH_NAME}-${COMMIT_SHA}
                        """
                    }
                }
            }
        }
    }
}
