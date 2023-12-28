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
                    // Upload the contents of the repository to the root of the S3 bucket
                    sh "aws s3 sync . s3://${BucketName}/${S3BucketPath} --region ${Region}"
                }
            }
        }

        stage('Create Beanstalk Application Version') {
            steps {
                script {
                    // Create an application version
                    scriptResult = sh(script: "aws elasticbeanstalk create-application-version --application-name '${ApplicationName}' --version-label '${BuildName}' --description 'Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}' --region ${Region}", returnStatus: true)
                    if (scriptResult != 0) {
                        error "Failed to create Beanstalk application version"
                    }
                }
            }
        }

        stage('Deploy to Beanstalk Environment') {
            steps {
                script {
                    // Deploy the application version to the Beanstalk environment
                    scriptResult = sh(script: "aws elasticbeanstalk update-environment --environment-name '${EnvironmentName}' --version-label '${BuildName}' --region ${Region}", returnStatus: true)
                    if (scriptResult != 0) {
                        error "Failed to update Beanstalk environment"
                    }
                }
            }
        }
    }
}
