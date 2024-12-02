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
    stages {
        stage('Display Latest Tag') {
            steps {
                container('git') {
                    script {
                        def latestTag = sh(
                            script: '''
                            git fetch --tags
                            git tag --list | sort -V | tail -n 1
                            ''',
                            returnStdout: true
                        ).trim()
                        echo "Latest Tag: ${latestTag}"
                    }
                }
            }
        }
    }
}
