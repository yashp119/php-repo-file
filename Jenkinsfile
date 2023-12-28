pipeline {
    agent any

    environment {
        AWS_ACCESS_KEY_ID     = credentials('aws-access-key-id')
        AWS_SECRET_ACCESS_KEY = credentials('aws-secret-access-key')
        Region                = 'us-east-1'
        BucketName            = 'php-bucket11'
        BuildName             = "version-${BUILD_NUMBER}"
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
                    def ZipFileName = "${BuildName}.zip"
                    echo "Uploading ${ZipFileName} to S3 bucket: ${BucketName} path: ${S3BucketPath}/${ZipFileName}"
                    
                    // Check if the file exists before uploading
                    if (fileExists(ZipFileName)) {
                        sh "aws s3 cp ${ZipFileName} s3://${BucketName}/${S3BucketPath}/${ZipFileName} --region ${Region}"
                    } else {
                        error "The file ${ZipFileName} does not exist. Make sure the file is generated or provide the correct filename."
                    }
                }
            }
        }

        stage('Create Beanstalk Application Version') {
            steps {
                // Your steps for creating Beanstalk Application Version
            }
        }

        stage('Update Beanstalk Environment') {
            steps {
                // Your steps for updating Beanstalk Environment
            }
        }
    }
}
