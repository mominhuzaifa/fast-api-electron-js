pipeline {
    agent {
        docker {
            // This handles the entire runtime environment natively
            image 'electronuserland/builder:wine-chrome'
            reuseNode true
        }
    }

    environment {
        ARTIFACT_DIR  = 'dist'
        PYTHON_DIST   = 'backend/dist'
        WINEDEBUG     = '-all'
    }

    stages {
        stage('Clean & Setup') {
            steps {
                echo 'Wiping old workspaces to protect against node_modules corruption...'
                cleanWs()
            }
        }

        stage('Build Windows Binary (Python)') {
            steps {
                echo 'Compiling Python FastAPI Backend to Windows EXE...'
                // Install backend dependencies inside Wine and freeze the app
                sh '''
                    wine pip install -r backend/requirements.txt
                    wine pyinstaller --onefile --windowed --name=api backend/src/api.py --distpath ./${PYTHON_DIST}
                '''

                echo 'Placing Python Binary into Electron Assets...'
                sh '''
                    mkdir -p backend/bin/win
                    cp ${PYTHON_DIST}/api.exe backend/bin/win/
                '''
            }
        }

        stage('Package App (Electron)') {
            steps {
                echo 'Building Final Windows Installer via Electron Builder...'
                // Install frontend dependencies and compile the windows installer
                sh '''
                    npm install
                    npx electron-builder --win --x64
                '''
            }
        }

        stage('Archive Outputs') {
            steps {
                echo 'Archiving build artifacts...'
                archiveArtifacts artifacts: "${ARTIFACT_DIR}/*.exe", allowEmptyArchive: false
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution complete.'
        }
    }
}
