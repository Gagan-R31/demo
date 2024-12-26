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
                            kubectl get deployment ${DEPLOYMENT_NAME} -n ${NAMESPACE} > /dev/null 2>&1 && echo "true" || echo "false"
                            """,
                            returnStdout: true
                        ).trim()

                        if (deploymentExists == "true") {
                            echo "Deployment ${DEPLOYMENT_NAME} exists. Updating image..."
                            // Update image and validate deployment
                            sh """
                            kubectl set image deployment/${DEPLOYMENT_NAME} ${CONTAINER_NAME}=${DOCKERHUB_REPO}:latest -n ${NAMESPACE}
                            kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE} --timeout=60s
                            """
                        } else {
                            echo "Deployment ${DEPLOYMENT_NAME} does not exist. Creating a new deployment..."
                            // Create a new deployment
                            sh """
                            kubectl create deployment ${DEPLOYMENT_NAME} \
                                --image=${DOCKERHUB_REPO}:latest \
                                --namespace=${NAMESPACE}
                            kubectl rollout status deployment/${DEPLOYMENT_NAME} -n ${NAMESPACE} --timeout=60s
                            """
                        }
                    }
                }
            }
        }
    }
}
