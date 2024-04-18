pipeline {
  agent any

  stages {
    stage('Build') {
      steps {
        sh 'docker build -f Dockerfile.prod -t $DOCKER_IMAGE_FE .'
      }
    }
    stage('Deploy') {
      steps {
        withCredentials([usernamePassword(credentialsId: "${DOCKER_REGISTRY_CREDS}", passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
          sh "echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin docker.io"
          sh 'docker push $DOCKER_IMAGE_FE'
        }
      }
    }
    stage('Build on instance') {
      steps {
        sshagent(credentials: ['ssh-cred']) {
            sh """
                ssh -o StrictHostKeyChecking=no ec2-user@54.66.35.172 '
                    if docker ps | grep -q lulfe; then
                      docker stop lulfe
                    else
                      echo "Container is not running."
                    fi                    
                    docker pull tienanhknock/lulfrontend
                    docker run --name lulfe -d -p 3000:80 --rm -e REACT_APP_BE_HOST=54.66.35.172 tienanhknock/lulfrontend
                '
            """
        }
      }
    }
  }
  post {
    always {
      sh 'docker logout'
    }
  }
}