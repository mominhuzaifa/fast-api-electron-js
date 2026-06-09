pipeline {
    agent {
        node {
            label 'linux-slave'
        }
    }

    environment {
        WINEDEBUG     = '-all'
    }

    stages {
        stage('Build & Package App') {
            steps {
                echo 'Creating required asset compilation directories on host...'
                sh 'mkdir -p backend/bin/win'

                echo 'Running Combined Python & Electron Pipeline inside PyInstaller Container...'
                sh '''
                    docker run --rm \
                        -v "${WORKSPACE}":/src \
                        -w /src \
                        cdrx/pyinstaller-windows:python3 \
                        sh -c "
                            echo '=== Step 1: Compiling Python Backend ===' && \
                            pip install -r backend/requirements.txt && \
                            pyinstaller --onefile --windowed --name=api backend/src/api.py --distpath ./backend/bin/win && \
                            
                            echo '=== Step 2: Packaging Electron App ===' && \
                            npm install && \
                            npx electron-builder --win --x64 && \
                            
                            echo '=== Step 3: Checking Generated Folders Inside Container ===' && \
                            ls -la
                        "
                '''

                echo '=== Step 4: Reclaiming Workspace Permissions on Host ==='
                sh 'sudo chown -R $(id -u):$(id -g) "${WORKSPACE}" || true'
                
                echo '=== Step 5: Locating Final Installer on Host Workspace ==='
                sh 'find . -name "*.exe" -maxdepth 3'
            }
        }

        stage('Archive Outputs') {
            steps {
                echo 'Archiving build artifacts natively...'
                // This relaxed double-asterisk scan catches your .exe anywhere in the workspace directory tree safely
                archiveArtifacts artifacts: "**/*.exe", allowEmptyArchive: false
            }
        }
    }

    post {
        cleanup {
            sh 'sudo chown -R $(id -u):$(id -g) "${WORKSPACE}" || true'
            cleanWs()
        }
    }
}
