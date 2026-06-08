pipeline {
    // Force the initial checkout and build context to instantiate on your AWS agent immediately
    agent {
        label 'linux-slave'
    }

    environment {
        ARTIFACT_DIR  = 'dist'
        PYTHON_DIST   = 'backend/dist'
        WINEDEBUG     = '-all'
    }

    stages {
        stage('Build Windows Binary (Python)') {
            agent {
                docker {
                    image 'cdrx/pyinstaller-windows:python3'
                    // Reuses the instance workspace already created by the parent agent block
                    reuseNode true
                    args '-u root:root'
                }
            }
            steps {
                echo 'Compiling Python FastAPI Backend to Windows EXE...'
                sh '''
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
                    image 'electronuserland/builder:20'
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
            // Reuses the parent agent node cleanly to access the generated executable
            steps {
                echo 'Archiving build artifacts...'
                archiveArtifacts artifacts: "${env.ARTIFACT_DIR}/*.exe", allowEmptyArchive: false
            }
        }
    }

    post {
        success {
            echo 'Windows package generated successfully!'
        }
        cleanup {
            // Executed properly now that a persistent node context exists globally
            cleanWs()
        }
    }
}
