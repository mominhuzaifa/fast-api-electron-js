pipeline {
    agent {
        docker {
            image 'electronuserland/builder:wine-chrome'
            reuseNode true
            args '--entrypoint=""'
        }
    }

    environment {
        ARTIFACT_DIR  = 'dist'
        PYTHON_DIST   = 'backend/dist'
        WINEDEBUG     = '-all'
        WINEPREFIX    = "${WORKSPACE}/.wine"
        // FIX 1: Use the precise value 'win64' expected by Wine
        WINEARCH      = 'win64'
        // FIX 2: Disable interactive GUI popups for missing engines in headless build
        WINEDLLOVERRIDES = "mscoree,mshtml="
    }

    stages {
        stage('Build Windows Binary (Python)') {
            steps {
                echo 'Compiling Python FastAPI Backend to Windows EXE...'
                sh '''
                    mkdir -p ${WINEPREFIX}
                    xvfb-run --server-args="-screen 0 1024x768x24" wine64 python -m pip install --upgrade pip
                    xvfb-run --server-args="-screen 0 1024x768x24" wine64 python -m pip install -r backend/requirements.txt
                    xvfb-run --server-args="-screen 0 1024x768x24" wine64 pyinstaller --onefile --windowed --name=api backend/src/api.py --distpath ./${PYTHON_DIST}
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

