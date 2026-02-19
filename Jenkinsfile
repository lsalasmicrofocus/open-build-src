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

                    // Download Debricked CLI tar.gz
                    bat """
                        curl -L -o debricked.tar.gz https://github.com/debricked/cli/releases/latest/download/cli_${osName}_${osArch}.tar.gz
                        
                        REM Try Git for Windows tar first
                        where tar >nul 2>nul && tar -xzf debricked.tar.gz || (
                            REM If tar not found, try 7-Zip
                            where 7z >nul 2>nul && 7z x debricked.tar.gz -y || (
                                REM If 7z not found, fallback to PowerShell
                                powershell -Command "tar -xzf debricked.tar.gz"
                            )
                        )
                    """

                    // Run the scan
                    bat 'debricked.exe scan'
                }
            }
        }
    }
}
