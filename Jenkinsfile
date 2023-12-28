pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        AWS_S3_BUCKET = 'php-bucket11'
        AWS_EB_APP_NAME = 'php-testing-app'
        AWS_EB_ENVIRONMENT = 'Php-testing-app-env'
        ARTIFACT_NAME = 'your-app.zip'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build and Package') {
            steps {
                script {
                    // Assuming you have a PHP project with a composer.json file
                    sh 'composer install'
                    
                    // Customize the packaging based on your PHP project structure
                    sh 'zip -r $ARTIFACT_NAME .'
                }
                archiveArtifacts artifacts: "**/${ARTIFACT_NAME}", fingerprint: true
            }
        }

        stage('Upload to S3') {
            steps {
                script {
                    sh "aws configure set region ${AWS_REGION}"
                    sh "aws s3 cp ./${ARTIFACT_NAME} s3://${AWS_S3_BUCKET}/${ARTIFACT_NAME}"
                }
            }
        }

        stage('Create Beanstalk Application Version') {
            steps {
                script {
                    sh "aws elasticbeanstalk create-application-version --application-name ${AWS_EB_APP_NAME} --version-label ${BUILD_NUMBER} --description 'Build created from JENKINS. Job:${JOB_NAME}, BuildId:${BUILD_DISPLAY_NAME}, GitCommit:${GIT_COMMIT}, GitBranch:${GIT_BRANCH}' --source-bundle S3Bucket=${AWS_S3_BUCKET},S3Key=${ARTIFACT_NAME}"
                }
            }
        }

        stage('Update Beanstalk Environment') {
            steps {
                script {
                    sh "aws elasticbeanstalk update-environment --application-name ${AWS_EB_APP_NAME} --environment-name ${AWS_EB_ENVIRONMENT} --version-label ${BUILD_NUMBER}"
                }
         }
        }
    }


}
