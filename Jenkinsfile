pipeline {
    agent {
        docker {
            image 'electronuserland/builder:wine-chrome'
            reuseNode true
            // Forces an interactive TTY and allocations that prevent shell spawn timing out
            args '-v /var/run/docker.sock:/var/run/docker.sock --entrypoint=""'
        }
    }

    environment {
        ARTIFACT_DIR  = 'dist'
        PYTHON_DIST   = 'backend/dist'
        WINEDEBUG     = '-all'
    }

    stages {
        stage('Build Windows Binary (Python)') {
            steps {
                echo 'Compiling Python FastAPI Backend to Windows EXE...'
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
        success {
            echo 'Windows package generated successfully!'
        }
        cleanup {
            // Safe post-execution cleanup running on the host workspace block
            cleanWs()
        }
    }
}
