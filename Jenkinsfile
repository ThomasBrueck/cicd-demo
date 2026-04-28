pipeline {
    // Se ejecutará en cualquier agente (o nodo) disponible en Jenkins
    agent any 

    stages {
        stage('Checkout') {
            steps {
                // Clona el código fuente desde el repositorio configurado
                checkout scm
            }
        }

        stage('Build') {
            steps {
                // Compila la aplicación Java. 
                // Nota: Usamos el comando "make build" que ya tienes en tu proyecto
                // porque automáticamente levanta un contenedor con Maven sin tener
                // que instalar Maven a mano en tu servidor Jenkins.
                sh 'make build'
            }
        }

        stage('Docker Build') {
            steps {
                // Construye la imagen Docker a partir de tu Dockerfile
                sh 'docker build -t mi-app:latest .'
            }
        }

        stage('Test') {
            steps {
                // Realiza las pruebas unitarias usando nuevamente la abstracción del Makefile
                // para que use el entorno de contenedores automático.
                sh 'make integrationTest'
            }
        }
    }
}
