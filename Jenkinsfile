pipeline {
  agent {
    node {
      label 'agent-dind'
    }

  }
  stages {
    stage('Checkout SCM') {
      steps {
        sh 'git checkout $BRANCH_NAME'
        sh 'git pull'
      }
    }

    stage('Build Feature') {
      steps {
        sh "docker build --tag $REPO_URL/$IMAGE_NAME:tmp ."
      }
    }

    stage('Upload Feature') {
      when {
        not {
          branch 'master'
        }

      }
      steps {
        sh "docker tag $REPO_URL/$IMAGE_NAME:tmp $REPO_URL/$IMAGE_NAME:$TAG_NAME"
        sh "docker push $REPO_URL/$IMAGE_NAME:$TAG_NAME"
      }
    }

    stage('Upload Release') {
      when {
        branch 'master'
      }
      steps {
        sh "docker tag $REPO_URL/$IMAGE_NAME:tmp $REPO_URL/$IMAGE_NAME:$BUILD_NUMBER"
        sh "docker tag $REPO_URL/$IMAGE_NAME:tmp $REPO_URL/$IMAGE_NAME:latest"
        sh "docker tag $REPO_URL/$IMAGE_NAME:tmp $REPO_URL/$IMAGE_NAME:latest"
        sh "docker push $REPO_URL/$IMAGE_NAME:$BUILD_NUMBER"
        sh "docker push $REPO_URL/$IMAGE_NAME:latest"
        sh "git tag master-$BUILD_NUMBER"
        sh 'git push --tags'
      }
    }

  }
  environment {
    REPO_URL = "$repository"
    IMAGE_NAME = "${imageName == 'FROM_GIT_REPO' ? determineRepoName() : $imageName}"
    TAG_NAME = "$tagName".replace("feature/","")
  }
}