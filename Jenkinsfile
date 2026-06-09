pipeline {
    agent {
        node {
            label 'linux-slave'
        }
    }

    environment {
        ARTIFACT_DIR  = 'dist'
        WINEDEBUG     = '-all'
    }

    stages {
        stage('Build Windows Binary (Python)') {
            steps {
                echo 'Compiling Python FastAPI Backend directly to the correct Electron Asset directory...'
                
                // Set the workdir inside the container to /src and force --distpath to point directly to backend/bin/win
                sh '''
                    docker run --rm \
                        -v "${WORKSPACE}":/src \
                        -w /src \
                        cdrx/pyinstaller-windows:python3 \
                        sh -c "pip install --upgrade pip && pip install -r backend/requirements.txt && pyinstaller --onefile --windowed --name=api backend/src/api.py --distpath ./backend/bin/win"
                '''

                echo 'Verifying compilation output location...'
                sh 'ls -la backend/bin/win/'
            }
        }

        stage('Package App (Electron)') {
            steps {
                echo 'Building Final Windows Installer via Electron Builder Container...'
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
