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
    stage('Build & Unit Tests') {
      steps {
        bat 'mvn -B -Dmaven.test.failure.ignore=false clean test'
      }
    }
    stage('Checkstyle') {
      steps {
        bat 'mvn -B -q checkstyle:check checkstyle:checkstyle'
      }
    }
    stage('JaCoCo Coverage') {
      steps {
        bat 'mvn -B -q verify'
      }
      post {
        always {
          archiveArtifacts artifacts: 'target/site/jacoco/**', fingerprint: true
        }
      }
    }
    stage('Package .jar') {
      steps {
        bat 'mvn -B -DskipTests package'
      }
    }
    // Subir artifact a Github Packages
    stage('Publish Artifact') {
      steps {
        bat 'mvn -B -Pgithub "-Dgpr.owner=chapaxD" "-Dgpr.repo=spring-docker" -DskipTests deploy'
      }
    }
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
    stage('Deploy to Staging via SSH') {
      when { expression { return env.STAGING_HOST != null } }
      steps {
        withCredentials([sshUserPrivateKey(credentialsId: 'staging_ssh_key', keyFileVariable: 'KEYFILE', usernameVariable: 'SSHUSER'), string(credentialsId: 'staging_host', variable: 'STAGING_HOST')]) {
          bat 'pscp -i %KEYFILE% -batch target\\*.jar %SSHUSER%@%STAGING_HOST%:/opt/apps/demo/demo.jar'
          bat 'plink -i %KEYFILE% -batch %SSHUSER%@%STAGING_HOST% "sudo systemctl restart demo.service || (pkill -f demo.jar; nohup java -jar /opt/apps/demo/demo.jar >/opt/apps/demo/app.log 2>&1 &)"'
        }
      }
    }
    stage('Staging Health Check') {
      when { expression { return env.STAGING_HOST != null } }
      steps {
        withCredentials([string(credentialsId: 'staging_host', variable: 'STAGING_HOST')]) {
          bat 'curl -fsS http://%STAGING_HOST%:8080/actuator/health | findstr UP'
        }
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
