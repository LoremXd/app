pipeline {
    agent {
        kubernetes {
            defaultContainer 'jenkins-worker'
            yaml """
apiVersion: v1
kind: Pod
metadata:
  name: kaniko
spec:
  nodeName: kube-node-1
  containers:
  - name: kaniko-frontend
    image: harbor.loremxd/containers/kaniko-executor:v1-debug
    securityContext:
      runAsUser: 0
    command:
    - /busybox/sleep
    args:
    - 99d
    tty: true
    volumeMounts:
      - name: docker-credentials
        mountPath: /kaniko/.docker
      - name: ca-certs
        mountPath: /kaniko/ssl/certs/additional-ca-cert-bundle.crt
        subPath: ca.crt
  - name: kaniko-backend
    image: harbor.loremxd/containers/kaniko-executor:v1-debug
    securityContext:
      runAsUser: 0
    command:
    - /busybox/sleep
    args:
    - 99d
    tty: true
    volumeMounts:
      - name: docker-credentials
        mountPath: /kaniko/.docker
      - name: ca-certs
        mountPath: /kaniko/ssl/certs/additional-ca-cert-bundle.crt
        subPath: ca.crt
  - name: trivy
    image: harbor.loremxd/containers/trivy:0.70.0
    securityContext:
      runAsUser: 1000
      runAsGroup: 1000
      fsGroup: 1000
    command:
    - cat
    tty: true
    volumeMounts:
      - name: docker-credentials
        mountPath: .docker
      - name: ca-certs
        mountPath: /etc/ssl/certs/ca.crt
        subPath: ca.crt
      - name: trivy-cache
        mountPath: /home/jenkins/agent/caches/trivy
  volumes:
  - name: docker-credentials
    secret:
      secretName: harbor-secret
      items:
      - key: .dockerconfigjson
        path: config.json
  - name: ca-certs
    secret:
      secretName: ca-certs-secret
      items:
      - key: ca.crt
        path: ca.crt
  - name: trivy-cache
    persistentVolumeClaim:
      claimName: trivy-cache-pvc
            """
        }
    }
    environment {
        BACKEND_IMAGE_NAME = "harbor.loremxd/app/backend"
        FRONTEND_IMAGE_NAME = "harbor.loremxd/app/frontend"
        TRIVY_CACHE_DIR = "/home/jenkins/agent/caches/trivy"
        TAG = "v${BUILD_NUMBER}"
    }
    stages {
        stage('Checkout') {
            steps {
                git url: 'git@github.com:LoremXd/app.git', branch: 'master', credentialsId: 'jenkins-ssh-key'
            }
        }
        stage('Build images') {
            parallel {
                stage('Build frontend image') {
                    steps {
                        container('kaniko-frontend') {
                            sh '''
                                /kaniko/executor \
                                    --dockerfile frontend/Dockerfile \
                                    --destination ${FRONTEND_IMAGE_NAME}:${TAG} \
                                    --context ${WORKSPACE}/frontend \
                                    --cache \
                            --cache-repo harbor.loremxd/app/frontend-cache
                            '''
                        }
                    }
                }
                stage('Build backend image') {
                    steps {
                        container('kaniko-backend') {
                            sh '''
                                /kaniko/executor \
                                    --dockerfile backend/Dockerfile \
                                    --destination ${BACKEND_IMAGE_NAME}:${TAG} \
                                    --context ${WORKSPACE}/backend \
                                    --cache \
                            --cache-repo harbor.loremxd/app/backend-cache
                            '''
                        }
                    }
                }
            }
        }
        stage('Scan images with Trivy') {
            steps {
                container('trivy') {
                    sh '''
                    mkdir -p reports
                    trivy image \
                      --exit-code 0 \
                      --severity CRITICAL,HIGH,MEDIUM \
                      --ignore-unfixed \
                      --format template \
                      --template @${TRIVY_CACHE_DIR}/html.tpl \
                      --cache-dir ${TRIVY_CACHE_DIR} \
                      --output ${WORKSPACE}/reports/trivy-report-frontend.html \
                        ${FRONTEND_IMAGE_NAME}:${TAG}
                    trivy image \
                      --exit-code 0 \
                      --severity CRITICAL,HIGH,MEDIUM \
                      --ignore-unfixed \
                      --format template \
                      --template @${TRIVY_CACHE_DIR}/html.tpl \
                      --cache-dir ${TRIVY_CACHE_DIR} \
                      --output ${WORKSPACE}/reports/trivy-report-backend.html \
                        ${BACKEND_IMAGE_NAME}:${TAG}
                    '''
                }
            }
        }
    }
    post {
        success {
            script {
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'trivy-report-*.html',
                    reportName: 'Trivy Vulnerability Report'
                ])
            }
        }
    }
}
