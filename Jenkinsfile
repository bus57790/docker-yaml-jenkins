pipeline {
  agent any

  environment {
    IMAGE_NAME = "docker-yaml-demo"
        IMAGE_TAG  = "latest"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Read YAML Config') {
      steps {
        script {
          def cfg = readYaml file: 'docker-build.yaml'

          // Expand ${BUILD_NUMBER} inside the YAML tag field
          def rawTag = cfg.image.tag.toString()
          def finalTag = rawTag.replace('${BUILD_NUMBER}', env.BUILD_NUMBER)

          env.IMAGE_NAME = cfg.image.name.toString()
          env.IMAGE_TAG  = finalTag

          env.BUILD_CONTEXT = cfg.build.context.toString()
          env.DOCKERFILE    = cfg.build.dockerfile.toString()

          env.REG_ENABLED = cfg.registry.enabled.toString()
          env.REG_URL     = cfg.registry.url.toString()
          env.REG_REPO    = cfg.registry.repository.toString()

          echo "IMAGE_NAME=${env.IMAGE_NAME}"
          echo "IMAGE_TAG=${env.IMAGE_TAG}"
          echo "DOCKERFILE=${env.DOCKERFILE}"
          echo "CONTEXT=${env.BUILD_CONTEXT}"
          echo "REG_ENABLED=${env.REG_ENABLED}"
        }
      }
    }

    stage('Docker Build') {
      steps {
        sh '''
          set -e
          docker version
          docker build -t ${IMAGE_NAME}:${IMAGE_TAG} -f ${DOCKERFILE} ${BUILD_CONTEXT}
          docker images | head -n 10
        '''
      }
    }

    stage('Docker Tag & Push (Optional)') {
      when {
        expression { return env.REG_ENABLED == "true" }
      }
      steps {
        sh '''
          set -e
          FULL_IMAGE=${REG_URL}/${REG_REPO}/${IMAGE_NAME}:${IMAGE_TAG}
          echo "Pushing: ${FULL_IMAGE}"

          docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${FULL_IMAGE}

          # If registry requires login, do: docker login ...
          docker push ${FULL_IMAGE}
        '''
      }
    }
  }

  post {
    success {
      echo "✅ Built image: ${IMAGE_NAME}:${IMAGE_TAG}"
    }
    always {
      sh 'docker system df || true'
    }
  }
}

