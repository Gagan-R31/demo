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
        stage('Check Tag or Branch') {
            steps {
                script {
                    if (buildingTags()) {
                        echo "Pipeline triggered by tag: ${env.GIT_TAG_NAME}"
                    } else {
                        echo "Pipeline triggered by branch: ${env.GIT_BRANCH}"
                    }
                }
            }
        }
    }
}
