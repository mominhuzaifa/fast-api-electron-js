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
                            npx electron-builder --win --x64
                        "
                '''

                echo '=== Step 3: Fixing Artifact Permissions on Host Agent ==='
                // This forces the host machine to reclaim ownership of everything the container created
                sh 'sudo chown -R $(id -u):$(id -g) "${WORKSPACE}/dist" || true'
                
                echo 'Verifying workspace dist folder contents:'
                sh 'ls -la dist/'
            }
        }

        stage('Archive Outputs') {
            steps {
                echo 'Archiving build artifacts...'
                archiveArtifacts artifacts: "dist/*.exe", allowEmptyArchive: false
            }
        }
    }

    post {
        success {
            echo 'Windows package generated successfully!'
        }
        cleanup {
            // Reclaim workspace permissions before cleanup to prevent ws-cleanup plugin from failing
            sh 'sudo chown -R $(id -u):$(id -g) "${WORKSPACE}" || true'
            cleanWs()
        }
    }
}


