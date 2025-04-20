pipeline {
  agent any

  environment {
    DOCKER_HUB_USER = 'sreenu112'
    DOCKER_IMAGE_NAME = 'maven-web-application'
    IMAGE_TAG = "${BUILD_NUMBER}"
    FULL_IMAGE = "${DOCKER_HUB_USER}/${DOCKER_IMAGE_NAME}:${IMAGE_TAG}"
    KUBE_NAMESPACE = 'dev'
    DEPLOYMENT_NAME = 'mavenwebappdeployment'
    CONTAINER_NAME = 'mavenwebapp'
    AWS_REGION = 'ap-south-1'
    CLUSTER_NAME = 'sampleEKS'
  }

  tools {
    maven 'maven3.9.9' // 👈 This name should match the one you configured in Jenkins Global Tools
  }

  stages {
    stage('Checkout') {
      steps {
        git url: 'https://github.com/sreenu11421/maven-web-application.git', branch: 'master'
      }
    }

    stage('Build Maven Project') {
      steps {
        sh 'mvn clean package'
      }
    }

    stage('Build Docker Image') {
      steps {
        sh "docker build -t ${FULL_IMAGE} ."
      }
    }

    stage('Push Docker Image to Docker Hub') {
      steps {
          //withCredentials([string(credentialsId: '1b5b8bda-e76d-4195-b0e3-5938644fb529', variable: 'dockerpassword')]) {
          //sh '''
           // echo "$dockerpassword" | docker login -u "$USERNAME" --password-stdin
          //  docker push ${FULL_IMAGE}
         // '''
        //}
        sh "docker login -u sreenu112 -p Ah50528@1142"
        sh "docker push ${FULL_IMAGE}"
      }
    }
    stage('Configure kubeconfig for EKS') {
      steps {
        withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
          sh '''
            aws eks update-kubeconfig --region ${AWS_REGION} --name ${CLUSTER_NAME}
          '''
        }
      }
    }
    stage('Update Kubernetes Deployment') {
      steps {
          sh """
          sed -i 's|image: .*|image: ${FULL_IMAGE}|' mavenwebappdeployment.yaml
          kubectl apply -f mavenwebappdeployment.yaml -n dev"""
      }
    }
  }

  post {
    success {
      echo "✅ Deployment to EKS using Docker Hub image successful!"
    }
    failure {
      echo "❌ Pipeline failed. Please check the logs."
    }
  }
}
