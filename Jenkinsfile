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
              - name: kubectl
                image: boxboat/kubectl
                command:
                - cat
                tty: true
              volumes:
              - name: workspace-volume
                emptyDir: {}
            """
        }
    }
    environment {
        DOCKERHUB_REPO = 'gaganr31/argu'
        DEPLOYMENT_NAME = 'browny'
        CONTAINER_NAME = 'browny'
        NAMESPACE = 'test-pipeline'
        
    }
    stages {
        stage('Deploy to K3s') {
            steps {
                container('kubectl') {
                    script {
                        // Check if the deployment exists
                        def deploymentExists = sh(
                            script: """
                            echo "false"
                            """
                        )
                    }
                }
            }
        }
    }
}
