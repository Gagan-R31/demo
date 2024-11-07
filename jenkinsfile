pipeline {
    agent {
        kubernetes {
            label 'k8s-agent'
            yaml '''
            apiVersion: v1
            kind: Pod
            metadata:
              labels:
                some-label: some-label-value
            spec:
              containers:
              - name: jnlp
                image: jenkins/inbound-agent
                args: ['$(JENKINS_SECRET)', '$(JENKINS_NAME)']
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
              volumes:
              - name: kaniko-secret
                secret:
                  secretName: kaniko-secret
                  items:
                  - key: .dockerconfigjson
                    path: config.json
              - name: workspace-volume
                emptyDir: {}
            '''
        }
    }
    environment {
        GITHUB_TOKEN = credentials('github-token') // Jenkins credentials ID for GitHub token
        DOCKERHUB_REPO = 'gaganr31/argu' // Your Docker Hub repository
        BUILD_TAG = "${env.BUILD_ID}" // Unique tag for each build
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    sh '''
                    git clone https://github.com/Gagan-R31/demo.git
                    cd demo
                    '''
                }
            }
        }
        stage('Build Docker Image with Kaniko') {
            steps {
                container('kaniko') {
                    script {
                        sh '''
                        cd demo
                            /kaniko/executor --dockerfile=./Dockerfile \
                                             --context=. \
                                             --destination=${DOCKERHUB_REPO}:${env.COMMIT_SHA}
                        '''
                    }
                }
            }
        }
    }
}
