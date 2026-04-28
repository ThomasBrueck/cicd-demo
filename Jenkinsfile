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
                // Le damos permiso de ejecución y le pedimos compilar el Java directamente.
                sh 'chmod +x ./mvnw'
                sh './mvnw clean package -DskipTests'
            }
        }

        stage('docker_build') {
            steps {
                // Docker sí funcionará excelentemente porque le pasamos el socket,
                // y como estamos construyendo *desde* Jenkins, lee los archivos sin problemas.
                sh 'docker build -t mi-app:latest .'
            }
        }

        stage('test') {
            steps {
                // Ejecutará las pruebas unitarias nativas
                sh './mvnw test -Dgroups=UnitTest'
            }
        }
    }
}
