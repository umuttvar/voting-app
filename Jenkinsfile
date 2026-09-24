pipeline {
    agent any 
    stages {
        stage('YAML dosyalarini listele') {
            steps {
                sh 'ls -la k8s/'
            }
        }
        stage('Bilgi ver') {
            steps {
                echo 'Voting app: 5 servis, Kubernetes uzerinde calisir'
            }
        }
    }
    post {
        success {
            echo 'Pipeline basariyla tamamlandi'
        }
        failure {
            echo 'Pipeline basarisiz oldu, loglara bak'
        }
    }
}
