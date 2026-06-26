pipeline {
    // Alternativa: agent { label 'docker' } — esegue la pipeline su un agent Jenkins con Docker
    // installato, utile per gli stage di build/push dell'immagine senza dipendere dal nodo master.
    agent any

    options {
        // Aggiunge il timestamp alle righe della console output della pipeline
        timestamps()
        /* Tutta la pipeline ha un timeout di 30 minuti */
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    parameters {
        booleanParam(
            name: 'SKIP_TESTS',
            defaultValue: false,
            description: 'Skip unit tests'
        )
        string(
            name: 'DOCKERHUB_USER',
            defaultValue: 'your-dockerhub-username',
            description: 'Username Docker Hub (es. mariorossi → mariorossi/sysfoo)'
        )
    }

    environment {
        IMAGE_NAME = "${params.DOCKERHUB_USER}/sysfoo"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm // Checkout the source code from the repository
            }
        }

        stage('Build') {
            steps {
                // Solo compilazione: fallisce in fretta su errori di sintassi
                sh 'mvn clean compile -DskipTests'
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

        stage('Package') {
            steps {
                // Crea il JAR solo dopo i test (o subito se SKIP_TESTS è attivo)
                sh 'mvn package -DskipTests'
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
                        # echo "$PASS" | docker login registry.example.com -u "$USER" --password-stdin
                        echo "$PASS" | docker login -u "$USER" --password-stdin
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
