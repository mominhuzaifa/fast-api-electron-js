pipeline {
    agent none 

    environment {
        ARTIFACT_DIR  = 'dist'
        PYTHON_DIST   = 'backend/dist'
        WINEDEBUG     = '-all'
    }

    stages {
        stage('Build Windows Binary (Python)') {
            agent {
                docker {
                    // This container already has Windows Python, Pip, and PyInstaller pre-installed inside Wine
                    image 'cdrx/pyinstaller-windows:python3'
                    reuseNode true
                }
            }
            steps {
                echo 'Compiling Python FastAPI Backend to Windows EXE...'
                sh '''
                    # Standard pip commands work instantly because the image handles the wine context natively
                    pip install --upgrade pip
                    pip install -r backend/requirements.txt
                    pyinstaller --onefile --windowed --name=api backend/src/api.py --distpath ./${PYTHON_DIST}
                '''

                echo 'Placing Python Binary into Electron Assets...'
                sh '''
                    mkdir -p backend/bin/win
                    cp ${PYTHON_DIST}/api.exe backend/bin/win/
                '''
            }
        }

        stage('Package App (Electron)') {
            agent {
                docker {
                    // Standard node container is fine here since the python binary is already built
                    image 'node:20-alpine'
                    reuseNode true
                }
            }
            steps {
                echo 'Building Final Windows Installer via Electron Builder...'
                sh '''
                    npm install
                    npx electron-builder --win --x64
                '''
            }
        }

        stage('Archive Outputs') {
            agent any
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
