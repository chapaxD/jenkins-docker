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
    // // Subir artifact a Github Packages
    // stage('Publish Artifact') {
    //   steps {
    //     bat 'mvn -B -Pgithub "-Dgpr.owner=chapaxD" "-Dgpr.repo=spring-docker" -DskipTests deploy'
    //   }
    // }
    // Buildar imagen Docker
    stage('Build Docker Image') {
      steps {
        bat 'docker build -t %IMAGE_NAME% .'
      }
    }
    // Ejecutar contenedor
    stage('Run Container') {
      steps {
        // Limpiar contenedor anterior
        bat 'docker rm -f demo-ci-cd || echo "No hay contenedor anterior"'
        
        // Ejecutar nuevo contenedor
        bat 'docker run -d --name demo-ci-cd -p %CONTAINER_PORT%:8080 %IMAGE_NAME%'
        
        // Verificar estado
        bat 'docker ps | findstr demo-ci-cd'
      }
    }
  }
  // Post-pipeline
  post {
    always {
      junit '**/target/surefire-reports/*.xml'
      archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    }
    // Si el pipeline termina exitosamente
    success {
      echo '🎉 Pipeline completado exitosamente!'
      echo '📱 Aplicación disponible en: http://localhost:%CONTAINER_PORT%'
    }
    // Si el pipeline termina con fallo
    failure {
      echo '❌ Pipeline falló!'
      bat 'docker ps -a | findstr demo-ci-cd || echo "No hay contenedores"'
    }
  }
}
