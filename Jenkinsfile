pipeline {
    agent any

    environment {
        BuildName = "version-${BUILD_NUMBER}"
        BucketName = "php-bucket11"
        ApplicationName = "php-testing-app"
        EnvironmentName = "Php-testing-app-env"
        MaxVersionsToKeep = 8
    }

    stages {
        stage('Build') {
            steps {
                script {
                    sh "zip -j ${BuildName}.zip ${env.WORKSPACE}/index.php"
                    sh "aws s3 cp ${BuildName}.zip s3://${BucketName} --region us-east-1"
                    sh "rm ${BuildName}.zip"
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

                    // Delete older versions
                    sh """
                        aws s3api list-object-versions --bucket ${BucketName} --prefix ${BuildName}.zip --region ap-south-1 |
                        jq -r '.Versions[:-${MaxVersionsToKeep}] | .[] | "DeleteMarker=\\(.DeleteMarker) Key=\\(.Key) VersionId=\\(.VersionId)"' |
                        xargs -I {} aws s3api delete-object --bucket ${BucketName} --region us-east-1 {}
                    """
                }
            }
        }
    }
}
