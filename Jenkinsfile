pipeline {
    agent {
        docker {
            image 'electronuserland/builder:wine-chrome'
            reuseNode true
            // Strip structural entrypoint configurations
            args '--entrypoint=""'
        }
    }

    environment {
        ARTIFACT_DIR  = 'dist'
        PYTHON_DIST   = 'backend/dist'
        WINEDEBUG     = '-all'
        // FIX 1: Force Wine to write its virtual C:\\ drive into our writable workspace
        WINEPREFIX    = "${WORKSPACE}/.wine"
    }

    stages {
        stage('Build Windows Binary (Python)') {
            steps {
                echo 'Compiling Python FastAPI Backend to Windows EXE...'
                // FIX 2: Initialize a headless virtual frame buffer wrapper (xvfb-run) 
                // so Wine can execute without XDG graphical environmental setups
                sh '''
                    mkdir -p ${WINEPREFIX}
                    xvfb-run --server-args="-screen 0 1024x768x24" wine pip install -r backend/requirements.txt
                    xvfb-run --server-args="-screen 0 1024x768x24" wine pyinstaller --onefile --windowed --name=api backend/src/api.py --distpath ./${PYTHON_DIST}
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
            cleanWs()
        }
    }
}
