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
    stages {
        stage('Load Pipeline Script') {
            steps {
                script {
                    if (env.BRANCH_NAME.startsWith('refs/tags/')) {
                        // Only trigger for tags that match the pattern "v*"
                        if (env.BRANCH_NAME.matches('^refs/tags/v.*')) {
                            echo "Loading tag-specific pipeline for: ${env.BRANCH_NAME}"
                            load 'tagPipeline.groovy'
                        } else {
                            echo "Tag does not match 'v*'. Skipping pipeline."
                            currentBuild.result = 'SUCCESS'
                        }
                    } else {
                        echo "Loading branch-specific pipeline for: ${env.BRANCH_NAME}"
                        load 'branchPipeline.groovy'
                    }
                }
            }
        }
    }
}
