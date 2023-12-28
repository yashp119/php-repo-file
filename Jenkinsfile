pipeline {
    agent any

    environment {
        BuildName = "version-${BUILD_NUMBER}"
        BucketName = "php-bucket11"
        ApplicationName = "php-testing-app"
        EnvironmentName = "Php-testing-app-env"
    }

    stages {
        stage('Build') {
            steps {
                script {
                    // Create a temporary directory to copy files to
                    def tempDir = "${env.WORKSPACE}/temp"
                    sh "mkdir -p ${tempDir}"

                    // Copy all files to the temporary directory
                    sh "cp -r ${env.WORKSPACE}/* ${tempDir}"

                    // Zip all files from the temporary directory
                    sh "cd ${tempDir} && zip -r ${BuildName}.zip ."

                    // Upload the zip file to S3
                    sh "aws s3 cp ${tempDir}/${BuildName}.zip s3://${BucketName} --region us-east-1"

                    // Remove the temporary directory
                    sh "rm -rf ${tempDir}"
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh """
                        aws elasticbeanstalk create-application-version \
                            --application-name "${ApplicationName}" \
                            --version-label "${BuildName}" \
                            --description "Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}" \
                            --source-bundle S3Bucket=${BucketName},S3Key=${BuildName}.zip \
                            --region us-east-1

                        aws elasticbeanstalk update-environment \
                            --environment-name "${EnvironmentName}" \
                            --version-label "${BuildName}" \
                            --region us-east-1
                    """
                }
            }
        }
    }
}
