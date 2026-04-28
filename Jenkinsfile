pipeline {
    agent any
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Build & Test') {
            steps {
                sh 'mvn clean package -DskipTests' // Compila y ejecuta la etapa UnitTest en Java
                sh 'mvn test -Dgroups=UnitTest'
            }
        }
        
        stage('Static Analysis (SonarQube)') {
            steps {
                script {
                    // El Plugin de Jenkins inyecta por detrás el Host URL y el Token
                    // 'sonarqube' debe ser el nombre del Server que definiste en Jenkins.
                    withSonarQubeEnv('sonarqube') {
                        sh 'mvn sonar:sonar -Dsonar.projectKey=my-app'
                    }
                }
            }
        }
        
        // **NUEVA ETAPA EXIGIDA POR EL TALLER:**
        stage('Quality Gate (SonarQube)') {
            steps {
                // Jenkins pausará hasta recibir la confirmación del Webhook.
                timeout(time: 15, unit: 'MINUTES') { 
                    // Si SonarQube halla un Hotspot (Regla Rota), aborta el Pipeline completo.
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Container Security Scan (Trivy)') {
            steps {
                sh 'docker build -t mi-app:latest .'
                
                // Si la prueba detecta riesgo 'CRITICAL', estalla la ejecución (Exit 1).
                sh 'trivy image --exit-code 1 --severity CRITICAL mi-app:latest'
            }
        }
        
        stage('Deploy') {
            when { branch 'master' } // Protege la ejecución a la rama master
            steps {
                sh 'docker rm -f mi-app-prod || true'
                sh 'docker run -d --name mi-app-prod -p 8081:8080 mi-app:latest'
            }
        }
    }
    
    post {
        always {
            echo 'Limpiando entorno y asegurando cierre...'
            // Limpia instancias remanentes con conflictos para que el host no explote a futuro.
            sh 'docker system prune -f'
            // Limpia el workspace clonado post-ejecución, devolviendo memoria.
            cleanWs() 
        }
    }
}
