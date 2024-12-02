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
              - name: curl-jq
                image: stedolan/jq
                command:
                - cat
                tty: true
                volumeMounts:
                - name: workspace-volume
                  mountPath: /workspace
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
        GITHUB_TOKEN = credentials('github-token-gagan') // GitHub token credential
        DOCKERHUB_REPO = 'gaganr31/argu'
        GITHUB_REPO = 'Gagan-R31/demo' // GitHub repo name
        DEPLOYMENT_NAME = 'argu'
        NAMESPACE = 'jenkins-operator'
    }
    stages {
        stage('Fetch Latest Release Tag') {
            steps {
                container('curl-jq') {
                    script {
                        withEnv(["GITHUB_AUTH=token ${GITHUB_TOKEN}"]) {
                            def response = sh(
                                script: """
                                curl -s -H "Authorization: ${GITHUB_AUTH}" \
                                https://api.github.com/repos/${GITHUB_REPO}/releases/latest | jq -r '.tag_name'
                                """,
                                returnStdout: true
                            ).trim()
                            
                            // Use the fetched tag as an environment variable
                            env.RELEASE_TAG = response
                            echo "Fetched GitHub release tag: ${RELEASE_TAG}"
                        }
                    }
                }
            }
        }
        stage('Clone Repository') {
            steps {
                script {
                    def workspaceDir = pwd()
                    sh """
                    git clone -b ${RELEASE_TAG} https://github.com/${GITHUB_REPO}.git
                    """
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
                                         --destination=${DOCKERHUB_REPO}:${RELEASE_TAG}
                        """
                    }
                }
            }
        }
    }
}
