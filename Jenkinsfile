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


