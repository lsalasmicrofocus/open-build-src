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

                    echo "OS detected: ${osName} and architecture ${osArch}"

                    // Download Debricked CLI zip using curl
                    bat """
                        curl -L -o debricked.zip https://github.com/debricked/cli/releases/latest/download/cli_${osName}_${osArch}.zip
                        powershell -Command "Expand-Archive -Path debricked.zip -DestinationPath . -Force"
                    """

                    // Run the scan with token
                    bat "debricked.exe scan --access-token %DEBRICKED_TOKEN%"
                }
            }
        }
    }
}
