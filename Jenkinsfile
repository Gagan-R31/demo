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
              - name: git
                image: alpine/git
                command:
                - cat
                tty: true
                volumeMounts:
                - name: workspace-volume
                  mountPath: /workspace
              volumes:
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
                    git clone --depth 1 --branch ${env.TAG_NAME} https://github.com/Gagan-R31/demo.git
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
                                         --destination=${DOCKERHUB_REPO}:${BRANCH_NAME}
                        """
                    }
                }
            }
        }
        stages {
        stage('Check Tag or Branch') {
            steps {
                script {
                   
                        echo "Pipeline triggered by tag: ${env.TAG_NAME}"
                  
                        echo "Pipeline triggered by branch: ${env.GIT_BRANCH}"
                    }
                }
            }
        }
    }


