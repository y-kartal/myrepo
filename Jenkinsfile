pipeline {
    agent any
    environment {
        DOCKER_IMAGE = 'y-kartal/myapp'  // DockerHub kullanıcı adına göre değiştir
        DOCKER_TAG = "v1.0.${env.BUILD_NUMBER}" // DOĞRU: Çift tırnak ve env.BUILD_NUMBER kullanılıyor
        DOCKER_REGISTRY_CREDENTIALS = 'dockerhub-credentials' // Jenkins'te tanımladığımız credential ID
        GIT_CREDENTIALS = 'github-credentials' // GitHub erişimi için tanımladığın credential ID
        HELM_CHART_NAME = 'my-helm-chart' // charts klasöründeki chart adı
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', credentialsId: "${GIT_CREDENTIALS}", url: 'https://github.com/y-kartal/myrepo.git'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', "${DOCKER_REGISTRY_CREDENTIALS}") {
                        def app = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                        app.push()
                    }
                }
            }
        }
        stage('Deploy to Kubernetes using Helm') {
            steps {
                script {
                    sh """
                    helm upgrade --install ${HELM_CHART_NAME} ./helm/${HELM_CHART_NAME} \
                        --set image.repository=${DOCKER_IMAGE} \
                        --set image.tag=${DOCKER_TAG} \
                        --namespace default
                    """
                }
            }
        }
    }
    post {
        success {
            echo '✅ Deployment completed successfully!'
        }
        failure {
            echo '❌ Deployment failed!'
        }
    }
}
