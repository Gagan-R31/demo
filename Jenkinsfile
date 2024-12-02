stages {
    stage('Clone Application Repository') { // Renamed
        steps {
            script {
                def workspaceDir = pwd()
                sh """
                git clone --depth 1 --branch ${env.TAG_NAME} ${env.REPO_URL}
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
                                     --destination=${DOCKERHUB_REPO}:${env.TAG_NAME}
                    """
                }
            }
        }
    }
    stage('Clone Helm Chart Repository') { // Renamed
        steps {
            script {
                def workspaceDir = pwd()
                sh """
                git clone ${HELM_CHART_REPO}
                """
            }
        }
    }
    stage('Update Helm Chart') {
        steps {
            container('yq') {
                script {
                    sh """
                    cd helm-chart
                    yq eval ".chartService.image.tag = \\"${env.TAG_NAME}\\"" -i helm-charts/values.yaml
                    """
                }
            }
        }
    }
    stage('Commit and Push Helm Chart Changes') {
        steps {
            container('git') {
                script {
                    sh """
                    cd helm-chart
                    git config user.name "Jenkins"
                    git config user.email "Gagan6696@gmail.com"
                    git add .
                    git commit -m "Update chartService image tag to ${env.TAG_NAME}"
                    git push origin master
                    """
                }
            }
        }
    }
}
