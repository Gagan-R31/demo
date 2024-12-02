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
        BRANCH_NAME = "${env.GIT_BRANCH}"
        TAG_NAME = "${env.GIT_TAG}"
    }

    stages {
        stage('Check Branch or Tag') {
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        echo "This is the main branch."
                        // Add steps for main branch here
                    } else if (env.TAG_NAME) {
                        echo "A tag has been pushed: ${TAG_NAME}"
                        // Add steps for tags here
                    } else {
                        echo "This is not the main branch or a tag."
                    }
                }
            }
        }
    }
} 
