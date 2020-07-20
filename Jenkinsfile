pipeline{
  agent any
  environment {
    registryBlue = "vinceangway/blue"
    registryGreen = "vinceangway/green"
    registryCredential = 'dockerhub'
    dockertag = getDockerTag()
  }
  stages{
    stage('Lint HTML'){
      steps{
            sh 'tidy -q -e Blue/index.html'
            sh 'tidy -q -e Green/index.html'
      }
    }
    stage("Lint Dockerfile"){
      agent{
          docker{
              image 'hadolint/hadolint:latest-debian'
          }
        }
      steps {
          sh 'hadolint /Blue/Dockerfile.blue | tee -a hadolint_lint.txt'
          sh '''
              lintErrors=$(stat --printf="%s"  hadolint_lint.txt)
              if [ "$lintErrors" -gt "0" ]; then
                  echo "Check Error"
                  cat hadolint_lint.txt
                  exit 1
              else
                  echo "No Error"
              fi
          '''
          sh 'hadolint /Green/Dockerfile.green | tee -a hadolint_lint.txt'
          sh '''
              lintErrors=$(stat --printf="%s"  hadolint_lint.txt)
              if [ "$lintErrors" -gt "0" ]; then
                  echo "Check Error"
                  cat hadolint_lint.txt
                  exit 1
              else
                  echo "No Error"
              fi
          '''
          }
        }
    stage('Build Docker Image'){
      steps{
        sh "docker build -f Blue/Dockerfile.blue Blue -t ${registryBlue}:${dockertag}"
        sh "docker build -f Green/Dockerfile.green Green -t ${registryGreen}:${dockertag}"
      }
    }
    stage('Push Docker Image'){
      steps{
        script{
          docker.withRegistry('', registryCredential) {
            sh "docker push ${registryBlue}:${dockertag}"
            sh "docker tag ${registryBlue}:${dockertag} ${registryBlue}:latest"
            sh "docker push ${registryBlue}:latest"
            sh "docker push ${registryGreen}:${dockertag}"
            sh "docker tag ${registryGreen}:${dockertag} ${registryGreen}:latest"
            sh "docker push ${registryGreen}:latest"
          }
        }
      }
    }

    stage('Deploying to EKS'){
      steps{
        withAWS(credentials: 'aws-credentials', region: 'us-west-2') {
          sh "aws eks --region us-west-2 update-kubeconfig --name Capstone"
          sh "kubectl apply -f myapp-blue.yml"
          sh "kubectl apply -f myapp-green.yml"
          sh "kubectl apply -f myapp-service.yml"
        }
      }
    }
  }    
}


def getDockerTag() {  
  def tag = sh script: 'git rev-parse --short=7 HEAD', returnStdout: true
  return tag.trim()
  }
