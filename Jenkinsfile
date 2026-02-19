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

                    // Download Debricked CLI ZIP
                    bat """
                        curl -L -o debricked.zip https://github.com/debricked/cli/releases/download/release-v2/cli_${osName}_${osArch}.zip
                        unzip -o debricked.zip
                    """

                    // Run the scan
                    bat 'debricked.exe scan'
                }
            }
        }
    }
}
