pipeline {
  agent any

  environment {
    AWS_REGION = 'ap-south-1'
    ECR_REPO = '967931774168.dkr.ecr.ap-south-1.amazonaws.com/sample-app'
    IMAGE_TAG = "v${BUILD_NUMBER}"
    GIT_REPO = 'https://github.com/Sana-786/sample-app.git'
    GIT_CREDENTIALS_ID = 'github-credentials'     // Jenkins credential ID for GitHub PAT
  }

  stages {

    stage('Checkout Code') {
      steps {
        git branch: 'main',
            credentialsId: "${GIT_CREDENTIALS_ID}",
            url: "${GIT_REPO}"
      }
    }

    stage('Build Docker Image') {
      steps {
        script {
          echo "🔧 Building Docker image..."
          sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
        }
      }
    }

    stage('Login to ECR & Push Image') {
      steps {
        script {
          echo "🔐 Logging into ECR..."
          sh """
            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
            docker push ${ECR_REPO}:${IMAGE_TAG}
          """
        }
      }
    }

    stage('Update K8s Manifest') {
      steps {
        script {
          echo "🧾 Updating Kubernetes manifest with new image tag..."
          sh """
            sed -i 's|image: .*|image: ${ECR_REPO}:${IMAGE_TAG}|g' manifests/deployment.yaml
          """
        }
      }
    }

    stage('Git Commit & Push Updated Manifest') {
      steps {
        script {
          echo "🪶 Committing updated manifest to GitHub..."
          sh """
            git config --global user.email "sanasiddiqui1999@gmail.com"
            git config --global user.name "sana siddiqui"
            git add manifests/deployment.yaml
            git commit -m "Update image tag to ${IMAGE_TAG}" || echo "No changes to commit"
          """
        }

        // ✅ Securely use Jenkins GitHub credentials for push
        withCredentials([usernamePassword(credentialsId: "${GIT_CREDENTIALS_ID}", usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
          sh '''
            git remote set-url origin https://${GIT_USER}:${GIT_TOKEN}@github.com/Sana-786/sample-app.git
            git push origin main
          '''
        }
      }
    }
  }

  post {
    success {
      echo "✅ Pipeline completed successfully. Argo CD will detect the change and sync it."
    }
    failure {
      echo "❌ Pipeline failed. Please check the logs."
    }
  }
}
