pipeline {
  agent any

  options {
    timestamps()  // built-in; no plugin dependencies
  }

  environment {
    // Debricked access token stored as a Jenkins "Secret text" credential
    DEBRICKED_TOKEN = credentials('DEBRICKED_TOKEN')
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Upgrade Gradle Wrapper to 8.14.4 (workspace-only)') {
      when {
        anyOf {
          expression { fileExists('gradlew') }
          expression { fileExists('gradlew.bat') }
        }
      }
      steps {
        script {
          if (isUnix()) {
            // UNIX path: swallow failure, keep the stage green
            sh '''
              set -e
              echo "Attempting Gradle wrapper upgrade to 8.14.4 (Unix, workspace-only)..."
              ./gradlew --no-daemon wrapper --gradle-version 8.14.4 || {
                echo "[WARN] Gradle wrapper upgrade failed; continuing (old Gradle may still fail on Java 21)."
              }
              echo "Wrapper version after upgrade attempt:"
              ./gradlew --version || true
            '''
          } else {
            // WINDOWS path: avoid parentheses pitfalls; always exit 0
            bat '''
              @echo on
              setlocal

              if not exist gradlew.bat goto no_gradle

              echo Attempting Gradle wrapper upgrade to 8.14.4 (Windows, workspace-only)...
              call gradlew.bat --no-daemon wrapper --gradle-version 8.14.4
              if errorlevel 1 (
                echo [WARN] Gradle wrapper upgrade failed; continuing (old Gradle may still fail on Java 21).
              ) else (
                echo Wrapper upgrade succeeded.
              )

              echo Wrapper version after upgrade attempt:
              call gradlew.bat --version || echo [WARN] Could not query wrapper version

              goto end

              :no_gradle
              echo No Gradle wrapper found; skipping upgrade.

              :end
              endlocal
              exit /b 0
            '''
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
                call gradlew.bat --version
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
        echo "NOTES:"
        echo " - Debricked CLI for Windows is shipped as .tar.gz; we extract debricked.exe with tar. (Official docs and releases confirm this format.)"
        echo " - Gradle 8.5+ runs on Java 21; this job upgrades the wrapper in the workspace to 8.14.4 so Debricked's Gradle init won't fail with 'major version 65'."
        echo "   For a permanent fix, commit the wrapper bump in your repository."
      }
    }
  }
}
