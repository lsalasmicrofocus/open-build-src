pipeline {
  agent any

  // Runs Gradle with JDK 17 to avoid "major version 65" (Java 21) errors on older Gradle wrappers.
  // Gradle 7.x can run fine on JDK 17; Gradle 8.5+ can run on Java 21 if/when you upgrade the wrapper.
  tools {
    jdk 'JDK17'   // <— configure this tool name in Jenkins
  }

  environment {
    DEBRICKED_TOKEN = credentials('DEBRICKED_TOKEN')
  }

  options {
    // Make console easier to read
    ansiColor('xterm')
    timestamps()
  }

  stages {
    stage('Checkout') {
      steps {
        // Declarative: Checkout SCM happens automatically when using Jenkinsfile from SCM,
        // but we keep an explicit stage for clarity.
        checkout scm
      }
    }

    stage('Download Debricked CLI') {
      steps {
        script {
          if (isUnix()) {
            // Linux/macOS path (binary name is "debricked")
            sh '''
              set -e
              rm -f debricked debricked.tar.gz
              # Latest release asset for Linux x86_64
              curl -fL -o debricked.tar.gz https://github.com/debricked/cli/releases/latest/download/cli_linux_x86_64.tar.gz
              tar -xzf debricked.tar.gz debricked
              chmod +x debricked
              ./debricked --version || true
            '''
          } else {
            // Windows path (binary name is "debricked.exe")
            bat '''
              @echo on
              del /f /q debricked.exe 2>nul
              del /f /q debricked.tar.gz 2>nul
              rem Latest release asset for Windows x86_64 (.tar.gz is the correct format)
              curl -fL -o debricked.tar.gz https://github.com/debricked/cli/releases/latest/download/cli_windows_x86_64.tar.gz
              tar -xzf debricked.tar.gz debricked.exe
              if not exist debricked.exe (
                echo Failed to extract debricked.exe
                exit /b 1
              )
              debricked.exe --version
            '''
          }
        }
      }
    }

    stage('Debricked Scan') {
      steps {
        script {
          if (isUnix()) {
            sh '''
              set -e
              echo "Running Debricked scan (Unix) with token from credentials..."
              ./debricked scan --access-token "$DEBRICKED_TOKEN"
            '''
          } else {
            bat '''
              @echo on
              echo Running Debricked scan (Windows) with token from credentials...
              debricked.exe scan --access-token %DEBRICKED_TOKEN%
            '''
          }
        }
      }
    }

    // Optional, but helpful if your repo includes a Gradle project:
    // This shows which Gradle wrapper (if any) is present and verifies the JDK used during the scan.
    stage('Environment Sanity (Optional)') {
      steps {
        script {
          if (isUnix()) {
            sh '''
              echo "JAVA_HOME=$JAVA_HOME"
              which java || true
              if [ -f "./gradlew" ]; then
                ./gradlew --version || true
              fi
            '''
          } else {
            bat '''
              @echo on
              echo JAVA_HOME=%JAVA_HOME%
              where java
              if exist gradlew.bat (
                gradlew.bat --version
              )
            '''
          }
        }
      }
    }
  }

  post {
    always {
      script {
        echo "Build finished. If you still see Gradle errors like 'Unsupported class file major version 65',"
        echo "either update the repo's Gradle wrapper to 8.5+ or keep running the job with JDK17 as configured above."
      }
    }
  }
}
