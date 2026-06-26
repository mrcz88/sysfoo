pipeline {
    agent { label 'docker' }

    options {
        // Aggiunge il timestamp alle righe della console output della pipeline
        timestamps()
        // Tutta la pipeline ha un timeout di 30 minuti
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    environment {
        IMAGE_NAME = 'registry.example.com/sysfoo'
    }

    parameters {
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Skip unit tests'
        )
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Test') {
            when {
                expression { return !params.SKIP_TESTS }
            }
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    retry(2) {
                        sh 'mvn test'
                    }
                }
            }
            post {
                always {
                    // Prova a pubblicare i report JUnit
                    junit 'target/surefire-reports/*.xml'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t $IMAGE_NAME:$BUILD_NUMBER .'
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker-registry-creds',
                    usernameVariable: 'USER',
                    passwordVariable: 'PASS'
                )]) {
                    sh '''
                        echo "$PASS" | docker login registry.example.com -u "$USER" --password-stdin
                        docker push $IMAGE_NAME:$BUILD_NUMBER
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                // Aggiorna il deployment Kubernetes con la nuova immagine e il relativo build number.
                // Kubernetes esegue poi il rollout; lo stato si verifica con: kubectl rollout status
                sh 'kubectl set image deployment/sysfoo sysfoo=$IMAGE_NAME:$BUILD_NUMBER'
            }
        }
    }

    post {
        success {
            echo 'Pipeline completata'
        }
        failure {
            echo 'Pipeline fallita'
        }
        always {
            cleanWs()
        }
    }
}
