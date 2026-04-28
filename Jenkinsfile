pipeline {
    agent any 

    stages {
        stage('checkout') {
            steps {
                checkout scm
            }
        }

        stage('build') {
            steps {
                // Compilación limpia y directa con Maven
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('docker_build') {
            steps {
                // Construcción de tu imagen Docker
                sh 'docker build -t mi-app:latest .'
            }
        }

        stage('test') {
            steps {
                // Ejecución de pruebas limpias
                sh 'mvn test -Dgroups=UnitTest'
            }
        }
    }
}
