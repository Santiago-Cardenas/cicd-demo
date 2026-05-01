pipeline {
    agent any

    tools { 
        maven 'maven3' 
    }

    stages {
        // --- PARTE 1: COMPILACION ---
        stage('Build') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }

        // --- PARTE 2: ANALISIS DE CALIDAD (SONAR) ---
        stage('SonarQube') {
            steps {
                sh "mvn sonar:sonar -Dsonar.projectKey=my-app -Dsonar.host.url=http://sonarqube:9000/ -Dsonar.login=sqp_ccb517f4c76df18cf4b227db8774cf4b9149da43 -Dsonar.qualitygate.wait=true"
            }
        }

        // --- PARTE 3: ESCANEO DE SEGURIDAD (TRIVY) ---
        stage('Seguridad') {
            steps {
                sh 'docker build -t mi-app:latest .'
                // El exit-code 1 bloquea el despliegue si hay vulnerabilidades criticas
                sh 'trivy image --severity CRITICAL --exit-code 1 mi-app:latest'
            }
        }

        // --- PARTE 4: DESPLIEGUE EN PUERTO 80 ---
        stage('Deploy') {
            steps {
                sh 'docker stop mi-app-container || true'
                sh 'docker rm mi-app-container || true'
                sh 'docker run -d --name mi-app-container -p 80:8080 mi-app:latest'
            }
        }
    }

    // --- MANEJO DE ERRORES Y LIMPIEZA ---
    post {
        always {
            cleanWs()
        }
        failure {
            echo 'Algo salio mal en el proceso, revisar logs de Jenkins'
        }
    }
}
