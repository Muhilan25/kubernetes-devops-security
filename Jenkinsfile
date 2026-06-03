pipeline {
  agent any

  environment {
    IMAGE_TAG = "V${BUILD_NUMBER}"
    SCANNER_HOME = tool 'sonar'
    ECR_REPO = "072583797351.dkr.ecr.ap-south-1.amazonaws.com"
  }

  stages {

      stage("git checkout") {
        steps {
          git branch: 'main', url: 'https://github.com/Muhilan25/kubernetes-devops-security.git'
        }
      }

      // stage("git leaks") {
      //   steps {
      //     sh 'gitleaks detect --report-format=json --report-path=gitleaks-report.json --exit-code=1'
      //   }
      // }
      stage('Build Artifact') {
            steps {
              sh "mvn clean package -DskipTests=true"
              archive 'target/*.jar' 
            }
        }

        stage("unit test-junit") {
          steps {
            sh "mvn test"
          }
        }

        stage('SonarQube') {
          steps {
              withSonarQubeEnv('sonarqube-scanner') {
                  sh '''
                      mvn sonar:sonar \
                      -Dsonar.projectKey=numeric-application \
                      -Dsonar.projectName=numeric-application
                  '''
              }
          }
        }

        stage("quality gates") {
          steps {
            timeout(time: 2, unit: 'MINUTES') {
               waitForQualityGate abortPipeline: false, credentialsId: 'sonar-cred'
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