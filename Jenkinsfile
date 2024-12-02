pipeline {
    agent any
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
