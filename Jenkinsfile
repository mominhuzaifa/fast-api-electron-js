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
                            set -e
                            echo '=== Step 1: Installing Python Dependencies ==='
                            pip install -r backend/requirements.txt || echo 'Pip finished with warnings'
                            
                            echo '=== Step 2: Compiling Python Backend via PyInstaller ==='
                            pyinstaller --onefile --windowed --name=api backend/src/api.py --distpath ./backend/bin/win
                            
                            echo '=== Step 3: Installing Node Modules ==='
                            npm install
                            
                            echo '=== Step 4: Packaging Electron App ==='
                            npx electron-builder --win --x64
                        "
                '''

                echo '=== Step 5: Reclaiming Workspace Permissions on Host ==='
                sh 'sudo chown -R $(id -u):$(id -g) "${WORKSPACE}" || true'
                
                echo '=== Step 6: Verifying Workspace Output Files ==='
                sh 'find . -name "*.exe" -maxdepth 3'
            }
        }

        stage('Archive Outputs') {
            steps {
                echo 'Archiving build artifacts natively...'
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
