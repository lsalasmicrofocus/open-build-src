pipeline {
    agent any
    
    environment {
        DEBRICKED_TOKEN = credentials('DEBRICKED_TOKEN')
    }

    stages {
        stage('Debricked Scan') {
            steps {
                script {
                    // Since you're on Windows, we can simplify OS detection
                    def osName = "windows"
                    def osArch = "x86_64" // adjust if needed (e.g., arm64)

                    println("OS detected: " + osName + " and architecture " + osArch)

                    // Download Debricked CLI using PowerShell
                    bat """
                        powershell -Command ^
                          Invoke-WebRequest -Uri https://github.com/debricked/cli/releases/download/release-v2/cli_${osName}_${osArch}.zip -OutFile debricked.zip
                        powershell -Command ^
                          Expand-Archive -Path debricked.zip -DestinationPath .
                    """

                    // Run the scan with the Windows executable
                    bat 'debricked.exe scan'
                }
            }
        }
    }
}
