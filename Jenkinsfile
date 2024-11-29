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
        IMAGE_TAG = "v${BUILD_NUMBER}"
        HARBOR_REPO = "harbor.zapto.org/browny/my-app"
        HELM_CHART_DIR = "helm"
        KUBE_NAMESPACE = "browny"
    }
    stages {
        stage('Clone Repository') {
            steps {
                script {
                    sh """
                    echo "Cloning repository..."
                    git clone -b dev https://github.com/Gagan-R31/demo.git 
                    """
                }
            }
        }
        stage('Build and Push Image') {
            steps {
                container('kaniko') {
                    sh """
                    echo "Building and pushing the Docker image to Harbor..."
                    /kaniko/executor \
                    --dockerfile=./Dockerfile \
                    --context=. \
                    --destination=${HARBOR_REPO}:${IMAGE_TAG}
                    """
                }
            }
        }

        stage('Update Helm Chart') {
            steps {
                container('yq') {
                    script {
                        sh """
                        echo "Cloning Helm chart repository..."
                        git clone https://github.com/Gagan-R31/demo.git helm-repo
                        cd helm-repo/${HELM_CHART_DIR}

                        echo "Updating Helm chart with new image tag..."
                        yq e '.image.tag = "${IMAGE_TAG}"' -i values.yaml

                        git config user.name "CI Bot"
                        git config user.email "gagankumar4882@gmail.com"
                        git commit -am "Update image tag to ${IMAGE_TAG}"
                        git push origin dev
                        """
                    }
                }
            }
        }
        stage('Deploy Using Helm') {
            steps {
                container('kubectl') {
                    sh """
                    echo "Deploying application with Helm..."
                    helm upgrade --install my-app ${HELM_CHART_DIR} \
                        --namespace ${KUBE_NAMESPACE} \
                        --set image.repository=${HARBOR_REPO} \
                        --set image.tag=${IMAGE_TAG} \
                        --atomic --wait
                    """
                }
            }
        }
    }
    post {
        success {
            echo "Pipeline executed successfully! Image: ${HARBOR_REPO}:${IMAGE_TAG}"
        }
        failure {
            echo "Pipeline failed! Check logs for details."
        }
    }
}
