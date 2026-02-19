pipeline {
    agent any
    
    environment {
        DEBRICKED_TOKEN = credentials('DEBRICKED_TOKEN')
    }

    stages {
        stage('Debricked Scan') {
            steps {
                script {
                    // Hardcode since Jenkins is running on Windows
                    def osName = "windows"
                    def osArch = "x86_64" // adjust if needed

                    println("OS detected: " + osName + " and architecture " + osArch)

                    // Download Debricked CLI using curl (bundled with Git for Windows)
                    bat """
                        curl -L -o debricked.zip https://github.com/debricked/cli/releases/download/release-v2/cli_${osName}_${osArch}.zip
                        tar -xf debricked.zip
                    """

                    // Run the scan with the Windows executable
                    bat 'debricked.exe scan'
                }
            }
        }
    }
}
