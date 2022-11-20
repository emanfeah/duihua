pipeline {
    agent any
    environment {
        AWS_S3_BUCKET = "emanealotaiefeah-project-s3bucket" // Change the name of the S3Bucket here to match the one in the aws-s3bucket Terraform Module ///
        ARTIFACT_NAME = "duihua.war"
        AWS_EB_APP_NAME = "app" // This have to match the app name in the aws-elasticbeanstalk-cloudfront Terraform Module 
        AWS_EB_APP_VERSION = "${BUILD_ID}"
        AWS_EB_ENVIRONMENT = "App-env-1" // This have to match the env name in the aws-elasticbeanstalk-cloudfront Terraform Module
        SONAR_IP = "54.80.4.251" // Change this IP to the ec2 IP Address outputted in the beginning (Sonarqube Server) ///
        SONAR_PROJECT = "duha" // Set your Sonarqube project name ///
        SONAR_TOKEN = "b872ce6c13adbc2c29849bdc6573fd65cd1a1364" // Set your Sonarqube Token ///
    }
    stages {
        stage('Validate') {
            steps {
                sh "mvn validate"

                sh "mvn clean"
            }
        }
        stage('Build') {
            steps {
                sh "mvn compile"
            }
        }
        stage('Test') {
            steps {
                sh "mvn test"
            }
        }

        stage('Quality Scan'){
            steps {
               sh '''
                    mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=$SONAR_PROJECT \
                    -Dsonar.host.url=http://$SONAR_IP:9000 \
                    -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }
        stage('Package') {
            steps {
                sh "mvn package"
            }
            post{
                success{
                    archiveArtifacts artifacts: 'target/*.war', followSymlinks: false
                }
            }
        }
        stage('Publish artifacts to S3 Bucket') {
            steps {
                sh "aws s3 cp ./target/*.war s3://$AWS_S3_BUCKET/$ARTIFACT_NAME"
            }
         }
        stage ("terraform init") {
            steps {
                sh ('terraform -chdir=Terraform/modules/aws-elasticbeanstalk-cloudfront init') 
            }
        }
        stage ("terraform apply elasticbeanstalk") {
            steps {
                sh ('terraform -chdir=Terraform/modules/aws-elasticbeanstalk-cloudfront apply -target="aws_elastic_beanstalk_application.$AWS_EB_APP_NAME" -target="aws_elastic_beanstalk_environment.$AWS_EB_ENVIRONMENT" --auto-approve')
           }
        }
        stage('Deploy') {
            steps {
                sh 'aws elasticbeanstalk create-application-version --application-name $AWS_EB_APP_NAME --version-label $AWS_EB_APP_VERSION --source-bundle S3Bucket=$AWS_S3_BUCKET,S3Key=$ARTIFACT_NAME'
                sh 'aws elasticbeanstalk update-environment --application-name $AWS_EB_APP_NAME --environment-name $AWS_EB_ENVIRONMENT --version-label $AWS_EB_APP_VERSION'
            }
         }
        // stage ("terraform apply cloudfront") {
        //     steps {
        //         sh ('terraform -chdir=Terraform/modules/aws-elasticbeanstalk-cloudfront destroy -target="aws_cloudfront_distribution.distribution" --auto-approve')
        //         sh ('terraform -chdir=Terraform/modules/aws-elasticbeanstalk-cloudfront apply -target="aws_cloudfront_distribution.distribution" --auto-approve')
        //    }
        // }
        

    }
}