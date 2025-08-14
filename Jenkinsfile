pipeline {
  agent any
  environment {
    IMAGE_NAME = "demo-ci-cd:latest"
    CONTAINER_PORT = "8081"
  }
  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }
    stage('Build & Test') {
      steps {
        bat 'mvn -B clean package'
      }
    }
    stage('Build Docker Image') {
      steps {
        bat 'docker build -t %IMAGE_NAME% .'
      }
    }
    stage('Run Container') {
      steps {
        // Detener contenedor existente si existe
        bat 'docker rm -f demo-ci-cd || exit 0'
        
        // Verificar que el puerto esté libre
        bat 'netstat -ano | findstr :%CONTAINER_PORT% || echo "Puerto %CONTAINER_PORT% disponible"'
        
        // Ejecutar contenedor en puerto alternativo
        bat 'docker run -d --name demo-ci-cd -p %CONTAINER_PORT%:8080 %IMAGE_NAME%'
        
        // Esperar un momento para que la aplicación se inicie
        bat 'timeout /t 10 /nobreak'
        
        // Verificar que el contenedor esté ejecutándose
        bat 'docker ps | findstr demo-ci-cd'
      }
    }
  }
  post {
    always {
      junit '**/target/surefire-reports/*.xml'
      archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    }
    success {
      echo '🎉 Pipeline completado exitosamente!'
      echo '📱 Aplicación disponible en: http://localhost:%CONTAINER_PORT%'
    }
    failure {
      echo '❌ Pipeline falló!'
      // Mostrar logs del contenedor si existe
      bat 'docker logs demo-ci-cd || echo "No se pudieron obtener logs"'
    }
  }
}
