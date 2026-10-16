pipeline {
  agent any
  environment {
    REGISTRY = 'crpi-5t4wtvylkfog8f2v.cn-hangzhou.personal.cr.aliyuncs.com'
    ACR_NS   = 'dtdev'
    APP      = 'test'
    IMAGE    = "${REGISTRY}/${ACR_NS}/${APP}"
    K8S_NS   = 'apps'
  }
  triggers { pollSCM('H/5 * * * *') }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Build') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'acr-creds', usernameVariable: 'ACR_USER', passwordVariable: 'ACR_PASS')]) {
          sh 'echo "$ACR_PASS" | docker login -u "$ACR_USER" --password-stdin "$REGISTRY"'
        }
        sh 'docker build -t "$IMAGE:$BUILD_NUMBER" -t "$IMAGE:latest" .'
      }
    }
    stage('Push') {
      steps {
        sh 'docker push "$IMAGE:$BUILD_NUMBER"'
        sh 'docker push "$IMAGE:latest"'
      }
    }
    stage('Deploy') {
      steps {
        sh 'kubectl apply -f k8s/'
        sh 'kubectl -n "$K8S_NS" set image deployment/"$APP" "$APP=$IMAGE:$BUILD_NUMBER"'
        sh 'kubectl -n "$K8S_NS" rollout status deployment/"$APP" --timeout=180s'
      }
    }
  }
  post {
    failure { sh 'kubectl -n "$K8S_NS" rollout undo deployment/"$APP" || true' }
  }
}
