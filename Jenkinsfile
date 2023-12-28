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

        stage('Package Code') {
            steps {
                script {
                    // Create a ZIP file of the code
                    sh "zip -r ${BuildName}.zip ."
                }
            }
        }

        stage('Upload to S3') {
            steps {
                script {
                    def S3BucketPath = "" // Empty string for the root of the bucket
                    // Upload the ZIP file to the S3 bucket
                    sh "aws s3 cp ${BuildName}.zip s3://${BucketName}/${S3BucketPath}/${BuildName}.zip --region ${Region}"
                }
            }
        }

        stage('Create Beanstalk Application Version') {
            steps {
                script {
                    // Create an application version using the ZIP file
                    sh "aws elasticbeanstalk create-application-version --application-name '${ApplicationName}' --version-label '${BuildName}' --source-bundle S3Bucket=${BucketName},S3Key=${BuildName}.zip --region ${Region}"
                }
            }
        }

        stage('Deploy to Beanstalk Environment') {
            steps {
                script {
                    // Deploy the application version to the Beanstalk environment
                    sh "aws elasticbeanstalk update-environment --environment-name '${EnvironmentName}' --version-label '${BuildName}' --region ${Region}"
                }
            }
        }
    }
}
