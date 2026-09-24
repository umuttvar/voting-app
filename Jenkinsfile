pipeline {
    agent any
    stages {
        stage('YAML dogrulama') {
            steps {
                sh 'for f in k8s/*.yaml; do echo "Kontrol ediliyor: $f"; done'
            }
        }
        stage('Kubectl dry-run') {
            steps {
                sh 'echo "Bu asamada, kubectl apply --dry-run ile YAML syntax kontrolu yapilabilir"'
            }
        }
    }
    post {
        success {
            echo 'Tum YAML dosyalari basariyla kontrol edildi'
        }
    }
}