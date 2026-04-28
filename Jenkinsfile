pipeline {
    agent any
    
    triggers {
        pollSCM('* * * * *')
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build & Test') {
            steps {
                sh 'mvn clean package -DskipTests'
                sh 'mvn test -Dgroups=UnitTest'
            }
        }
        
        stage('Static Analysis (SonarQube)') {
            steps {
                script {
                    withSonarQubeEnv('sonarqube') {
                        sh 'mvn sonar:sonar -Dsonar.projectKey=my-app'
                    }
                }
            }
        }
        
        stage('Quality Gate (SonarQube)') {
            steps {
                timeout(time: 15, unit: 'MINUTES') { 
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Container Security Scan (Trivy)') {
            steps {
                sh 'docker build -t mi-app:latest .'
                sh 'trivy image --exit-code 0 --severity CRITICAL mi-app:latest'
            }
        }
        
        stage('Deploy') {
            steps {
                sh 'docker rm -f mi-app-prod || true'
                sh 'docker run -d --name mi-app-prod -p 80:8080 mi-app:latest'
            }
        }
    }
    
    post {
        always {
            echo 'Limpiando entorno y asegurando cierre...'
            sh 'docker system prune -f'
            cleanWs() 
        }
        failure {
            echo '🚨 CRÍTICO: Pipeline abortado. Simulación de envío de alerta!'
        }
    }
}
