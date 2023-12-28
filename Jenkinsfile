pipeline {
    agent any

    environment {
        BucketName = 'php-bucket11'
        ApplicationName = 'php-testing-app'
        BuildName = 'your-version-label'
        S3BucketPath = '' // Empty string for the root of the bucket
        BeanstalkEnvironmentName = 'Php-testing-app-env' // Replace with your actual Beanstalk environment name
    }

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout code from Git repository
                    checkout scm
                }
            }
        }

        stage('Package and Upload to S3') {
            steps {
                script {
                    // Zip the contents of the repository
                    sh 'zip -r app.zip .'
                    
                    // Upload the zip file to S3 bucket
                    sh "aws s3 cp app.zip s3://${BucketName}/${S3BucketPath}app.zip --region us-east-1"
                }
            }
        }

        stage('Create Beanstalk Application Version') {
            steps {
                script {
                    // Create Beanstalk application version
                    sh "aws elasticbeanstalk create-application-version --application-name '${ApplicationName}' --version-label '${BuildName}' --description 'Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}' --region us-east-1"
                }
            }
        }

        stage('Update Beanstalk Environment') {
            steps {
                script {
                    // Update Beanstalk environment
                    sh "aws elasticbeanstalk update-environment --application-name '${ApplicationName}' --environment-name '${BeanstalkEnvironmentName}' --version-label '${BuildName}' --region us-east-1"
                }
            }
        }
    }

    post {
        always {
            // Clean up, delete the temporary zip file
            sh 'rm -f app.zip'
        }
    }
}
