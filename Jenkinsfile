pipeline {
    agent {
        node {
            label 'linux-slave' // Dynamically provisions your AWS instance
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
                echo 'Compiling Python FastAPI Backend to Windows EXE via Native Workspace Docker Call...'
                
                // Run the image explicitly as a runtime script command to keep streams intact
                sh '''
                    docker run --rm \
                        -v "${WORKSPACE}":/src \
                        cdrx/pyinstaller-windows:python3 \
                        sh -c "pip install --upgrade pip && pip install -r backend/requirements.txt && pyinstaller --onefile --windowed --name=api backend/src/api.py --distpath ./${PYTHON_DIST}"
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
                echo 'Building Final Windows Installer via Electron Builder Container...'
                
                // Similarly executing via an explicit runtime shell pass
                sh '''
                    docker run --rm \
                        -v "${WORKSPACE}":/project \
                        -w /project \
                        electronuserland/builder:20 \
                        sh -c "npm install && npx electron-builder --win --x64"
                '''
            }
        }

        stage('Archive Outputs') {
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
            cleanWs()
        }
    }
}
