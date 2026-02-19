pipeline {
    agent any
    
    environment {
        DEBRICKED_TOKEN = credentials('DEBRICKED_TOKEN')
    }

    stages {
        stage('Debricked Scan') {
            steps {
                script {
                    def osName = "windows"
                    def osArch = "x86_64"

                    println("OS detected: " + osName + " and architecture " + osArch)

                    // Download Debricked CLI zip for Windows
                    powershell """
                        Invoke-WebRequest -Uri https://github.com/debricked/cli/releases/latest/download/cli_${osName}_${osArch}.zip -OutFile debricked.zip
                        Expand-Archive debricked.zip -DestinationPath .
                    """

                    // Run the scan
                    bat 'debricked.exe scan'
                }
            }
        }
    }
}
