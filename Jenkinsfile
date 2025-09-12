pipeline {
  agent {
    kubernetes {
      label 'kubedeploy-agent'
      yaml '''
        apiVersion: v1
        kind: Pod
        spec:
          containers:
          - name: kubectl
            image: eclipsefdn/kubectl:okd-c1
            command:
            - cat
            tty: true
            resources:
              limits:
                cpu: 1
                memory: 1Gi
            volumeMounts:
            - mountPath: "/home/default/.kube"
              name: "dot-kube"
              readOnly: false
          - name: jnlp
            resources:
              limits:
                cpu: 1
                memory: 1Gi
          volumes:
          - name: "dot-kube"
            emptyDir: {}
      '''
    }
  }

  environment {
    APP_NAME = 'open-vsx-org'
    NAMESPACE = 'open-vsx-org'
    IMAGE_NAME = 'ghcr.io/eclipsefdn/redis-cluster-operator'
    IMAGE_TAG = sh(
      script: """
        if [ "${env.TAG_NAME}" = "" ]; then
          printf ${env.TAG_NAME}-${env.BUILD_NUMBER}
        else
          printf \$(git rev-parse --short ${env.GIT_COMMIT})-${env.BUILD_NUMBER}
        fi
      """,
      returnStdout: true
    )
  }

  options {
    buildDiscarder(logRotator(numToKeepStr: '10'))
    timeout(time: 30, unit: 'MINUTES')
  }

  stages {
    stage('Build docker image') {
      agent {
        label 'docker-build'
      }
      steps {
        sh '''
          docker build --pull -t ${IMAGE_NAME}:${IMAGE_TAG} .
        '''
      }
    }

    stage('Push docker image') {
      agent {
        label 'docker-build'
      }
      steps {
        withDockerRegistry([credentialsId: 'a56a2346-7fc5-4f91-a624-073197e5f5c8', url: 'https://ghcr.io/']) {
          sh '''
            docker push ${IMAGE_NAME}:${IMAGE_TAG}
          '''
        }
      }
    }


  post {
    always {
      sh 'docker logout ${REGISTRY} || true'
    }
  }
}

