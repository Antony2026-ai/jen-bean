pipeline {
    agent any

    environment {
        IMAGE_NAME = "dockerbean"
        VERSION = "${BUILD_NUMBER}"
        DOCKER_HUB_REPO = "kreajith2026/dockerbean"

        AWS_REGION = "us-east-1"
        EB_APP_NAME = "bean-first-proj"
        EB_ENV_NAME = "Bean-first-proj-env"

        S3_BUCKET = "jen-bean-deploy-2026"   
        ZIP_FILE = "deploy-${BUILD_NUMBER}.zip"
    }

    stages {

        

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${VERSION} ."
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-hub',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh 'echo "$PASS" | docker login -u "$USER" --password-stdin'
                }
            }
        }

        stage('Tag & Push Image') {
            steps {
                sh "docker tag ${IMAGE_NAME}:${VERSION} ${DOCKER_HUB_REPO}:${VERSION}"
                sh "docker push ${DOCKER_HUB_REPO}:${VERSION}"
            }
        }

        stage('Create Dockerrun File') {
            steps {
                sh """
                cat > Dockerrun.aws.json <<EOF
{
  "AWSEBDockerrunVersion": 1,
  "Image": {
    "Name": "${DOCKER_HUB_REPO}:${VERSION}",
    "Update": "true"
  },
  "Ports": [
    {
      "ContainerPort": 5000
    }
  ]
}
EOF
                """
            }
        }

        stage('Zip for Deployment') {
            steps {
                sh "zip ${ZIP_FILE} Dockerrun.aws.json"
            }
        }

        stage('Upload to S3') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds']]) {

                    sh """
                    aws s3 cp ${ZIP_FILE} s3://${S3_BUCKET}/${ZIP_FILE} --region ${AWS_REGION}
                    """
                }
            }
        }

        stage('Deploy to Beanstalk') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-creds']]) {

                    sh """
                    aws elasticbeanstalk create-application-version \
                        --application-name ${EB_APP_NAME} \
                        --version-label ${VERSION} \
                        --source-bundle S3Bucket=${S3_BUCKET},S3Key=${ZIP_FILE} \
                        --region ${AWS_REGION}

                    aws elasticbeanstalk update-environment \
                        --environment-name ${EB_ENV_NAME} \
                        --version-label ${VERSION} \
                        --region ${AWS_REGION}
                    """
                }
            }
        }
    }

    post {
        success {
            echo "Deployment SUCCESS 🚀"
        }
        failure {
            echo "Deployment FAILED ❌"
        }
    }
}
