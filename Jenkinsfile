pipeline {
  agent any
  environment {
    IMAGE_TAG = "V${BUILD_NUMBER}"
    ECR_REPO = "072583797351.dkr.ecr.ap-south-1.amazonaws.com"
  }

  stages {
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' //so that they can be downloaded later
            }
        }

        stage("unit test-junit and jacoco") {
          steps {
            sh "mvn test"
          }
          post {
            always {
              junit 'target/surefire-reports/*.xml'
              jacoco execPattern: 'target/jacoco.exec'
            }
          }
        }   

        stage("docker image") {
          steps {
              withCredentials([
                  aws(
                      accessKeyVariable: 'AWS_ACCESS_KEY_ID',
                      credentialsId: 'aws-cred',
                      secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
                  )
              ]) {
                  sh '''
                      aws ecr get-login-password --region ap-south-1 | \
                      docker login --username AWS --password-stdin \
                      072583797351.dkr.ecr.ap-south-1.amazonaws.com

                      docker build -t spring-app:${IMAGE_TAG} .

                      docker tag spring-app:${IMAGE_TAG} \
                      ${ECR_REPO}/spring-app:${IMAGE_TAG}

                      docker push \
                      ${ECR_REPO}/spring-app:${IMAGE_TAG}
                  '''
              }
          }
      }

    }
}