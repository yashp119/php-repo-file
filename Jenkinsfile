pipeline {
    agent any

    environment {
        BUILD_NAME = "Beanstalk-Code-Version-${BUILD_ID}"
        BUCKET_NAME = "php-bucket11"
        ApplicationName = "personal-testing"
        EnvironmentName = "Personal-testing-env"
    }

    stages {
        stage('Build') {
            steps {
                script {
                    sh "cd /var/lib/jenkins/workspace/php-pipeline/"
                    sh "zip -r ${BUILD_NAME}.zip ."
                    sh "ls -l ${BUILD_NAME}.zip"
                    sh "aws s3 cp ${BUILD_NAME}.zip s3://$BUCKET_NAME --region us-east-1"
                    sh "rm -rf ./* .git"
                    
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    sh """
                        aws elasticbeanstalk create-application-version \
                            --application-name "${ApplicationName}" \
                            --version-label "${BUILD_NAME}" \
                            --description "Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}" \
                            --source-bundle S3Bucket=${BUCKET_NAME},S3Key=${BUILD_NAME}.zip \
                            --region us-east-1

                        aws elasticbeanstalk update-environment \
                            --environment-name "${EnvironmentName}" \
                            --version-label "${BUILD_NAME}" \
                            --region us-east-1
                    """
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    // Specify the number of versions to keep
                    def versionsToKeep = 3

                    // Get the list of application versions
                    def versions = sh(script: "aws elasticbeanstalk describe-application-versions --application-name ${ApplicationName} --region us-east-1 --query 'ApplicationVersions[*].VersionLabel' --output text", returnStdout: true).trim().split()

                    // Sort versions in descending order
                    versions.sort { a, b -> b.compareTo(a) }

                    // Remove excess versions and corresponding artifacts from S3
                    for (int i = versionsToKeep; i < versions.size(); i++) {
                        // Print information for debugging
                        echo "Deleting version: ${versions[i]}"

                        // Delete the application version from Elastic Beanstalk
                        sh "aws elasticbeanstalk delete-application-version --application-name ${ApplicationName} --version-label ${versions[i]} --region us-east-1"

                        // Delete the corresponding artifact from S3
                        sh "aws s3 rm s3://${BUCKET_NAME}/${versions[i]}.zip --region us-east-1"

                        // If versioning is enabled, delete all versions of the object in S3
                        sh "aws s3api delete-object --bucket ${BUCKET_NAME} --key ${versions[i]}.zip --region us-east-1"

                        // Print information for debugging
                        echo "Deleted version: ${versions[i]}"
                    }
                }
            }
        }
    }
}
