pipeline {
  agent any

  // No 'tools { jdk ... }' and no 'ansiColor' to avoid missing-tool/plugin errors.
  options {
    timestamps()   // built-in and available in your instance per the valid options list
  }

  environment {
    // Debricked access token stored as a Jenkins "Secret text" credential
    DEBRICKED_TOKEN = credentials('DEBRICKED_TOKEN')

    // OPTIONAL: Point this to a Java 17 path on your agent to avoid Gradle 7.x vs Java 21 issues.
    // Example Windows:  C:\Program Files\Eclipse Adoptium\jdk-17
    // Example Linux:    /usr/lib/jvm/temurin-17
    JAVA17_HOME = ''   // <— leave empty if you don't have it; or set at the job level
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Prepare Java (optional JDK 17)') {
      steps {
        script {
          // If JAVA17_HOME is provided, prepend it to PATH and export JAVA_HOME.
          if (env.JAVA17_HOME?.trim()) {
            if (isUnix()) {
              withEnv(["JAVA_HOME=${env.JAVA17_HOME}", "PATH=${env.JAVA17_HOME}/bin:${env.PATH}"]) {
                sh '''
                  set -e
                  echo "Using JAVA_HOME=$JAVA_HOME"
                  java -version || true
                '''
              }
            } else {
              withEnv(["JAVA_HOME=${env.JAVA17_HOME}", "PATH=${env.JAVA17_HOME}\\bin;${env.PATH}"]) {
                bat '''
                  @echo on
                  echo Using JAVA_HOME=%JAVA_HOME%
                  java -version
                '''
              }
            }
          } else {
            // No JAVA17_HOME provided; just show current Java
            if (isUnix()) {
              sh '''
                echo "JAVA_HOME not provided; using system Java"
                echo "PATH=$PATH"
                java -version || true
              '''
            } else {
              bat '''
                @echo on
                echo JAVA_HOME not provided; using system Java
                echo PATH=%PATH%
                java -version
              '''
            }
          }
        }
      }
    }

    stage('Download Debricked CLI') {
      steps {
        script {
          if (isUnix()) {
            // Linux/macOS: download .tar.gz and extract 'debricked'
            sh '''
              set -e
              rm -f debricked debricked.tar.gz
              curl -fL -o debricked.tar.gz https://github.com/debricked/cli/releases/latest/download/cli_linux_x86_64.tar.gz
              tar -xzf debricked.tar.gz debricked
              chmod +x debricked
              ./debricked --version || true
            '''
          } else {
            // Windows: download .tar.gz and extract 'debricked.exe'
            bat '''
              @echo on
              del /f /q debricked.exe 2>nul
              del /f /q debricked.tar.gz 2>nul
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
              echo "Running Debricked scan (Unix)..."
              ./debricked scan --access-token "$DEBRICKED_TOKEN"
            '''
          } else {
            bat '''
              @echo on
              echo Running Debricked scan (Windows)...
              debricked.exe scan --access-token %DEBRICKED_TOKEN%
            '''
          }
        }
      }
    }

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
        echo "NOTE:"
        echo " - Debricked CLI Windows asset is .tar.gz, not .zip (we use tar -xzf to extract)."
        echo " - If you still see Gradle errors like 'Unsupported class file major version 65', either:"
        echo "     a) Set JAVA17_HOME to a JDK 17 on the agent (this pipeline will use it), or"
        echo "     b) Upgrade your repo's Gradle wrapper to 8.5+ so it can run on Java 21."
      }
    }
  }
}
