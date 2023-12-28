pipeline {
    agent any

    environment {
        BuildName = "version-${BUILD_NUMBER}"
        BucketName = "php-bucket11"
        ApplicationName = "php-testing-app"
        EnvironmentName = "php-testing-app-env"
        Region = "us-east-1"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Upload to S3') {
            steps {
                script {
                    def S3BucketPath = "" // Empty string for the root of the bucket
                    echo "Uploading ${BuildName}.zip to S3 bucket: ${BucketName} path: ${S3BucketPath}/${BuildName}.zip"
                    sh "aws s3 cp ${BuildName}.zip s3://${BucketName}/${S3BucketPath}/${BuildName}.zip --region ${Region}"
                }
            }
        }

        stage('Create Beanstalk Application Version') {
            steps {
                script {
                    echo "Creating Beanstalk application version ${BuildName}"
                    sh "aws elasticbeanstalk create-application-version --application-name '${ApplicationName}' --version-label '${BuildName}' --description 'Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}' --region ${Region}"
                }
            }
        }

        stage('Update Beanstalk Environment') {
            steps {
                script {
                    echo "Updating Beanstalk environment ${EnvironmentName} to version ${BuildName}"
                    sh "aws elasticbeanstalk update-environment --environment-name '${EnvironmentName}' --version-label '${BuildName}' --region ${Region}"
                }
            }
        }
    }
}
