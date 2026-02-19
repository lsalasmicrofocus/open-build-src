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

                    // Download Debricked CLI using certutil (built into Windows)
                    bat """
                        certutil -urlcache -split -f https://github.com/debricked/cli/releases/download/release-v2/cli_${osName}_${osArch}.zip debricked.zip
                    """

                    // Extract ZIP using built-in expand (works for CAB/ZIP)
                    bat """
                        expand debricked.zip -F:* .
                    """

                    // Run the scan
                    bat 'debricked.exe scan'
                }
            }
        }
    }
}
