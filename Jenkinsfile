pipeline {
    agent any

    environment {
        BuildName = "version-${BUILD_NUMBER}"
        BucketName = "php-bucket11"
        ApplicationName = "php-testing-app"
        EnvironmentName = "Php-testing-app-env"
    }

    stages {
        stage('Checkout') {
            steps {
                // Assuming you are using Git for version control
                checkout scm
            }
        }

        stage('Upload to S3') {
            steps {
                script {
                    def S3BucketPath = "" // Empty string for the root of the bucket
                    // Upload the contents of the repository to the root of the S3 bucket
                    sh "aws s3 sync . s3://${BucketName}/${S3BucketPath} --region us-east-1"
                }
            }
        }

        stage('Create Beanstalk Application Version') {
            steps {
                script {
                    sh "aws elasticbeanstalk create-application-version --application-name '${ApplicationName}' --version-label '${BuildName}' --description 'Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}' --region us-east-1"
                }
            }
        }

        stage('Update Beanstalk Environment') {
            steps {
                script {
                    sh "aws elasticbeanstalk update-environment --environment-name '${EnvironmentName}' --version-label '${BuildName}' --region us-east-1"
                }
            }
        }
    }
}
