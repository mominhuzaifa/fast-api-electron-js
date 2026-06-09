pipeline {
    agent {
        node {
            label 'linux-slave' // Spawns your AWS EC2 agent
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

                echo 'Running Full Windows Build Pipeline inside an Electron Wine Environment...'
                
                // Wrap the internal commands securely inside quotes so everything executes inside the container context
                sh '''
                    docker run --rm \
                        -v "${WORKSPACE}":/project \
                        -w /project \
                        electronuserland/builder:wine \
                        sh -c "
                            echo '=== Step 1: Compiling Python FastAPI Backend ===' && \
                            pip3 install -r backend/requirements.txt && \
                            pyinstaller --onefile --windowed --name=api backend/src/api.py --distpath ./backend/bin/win && \
                            
                            echo '=== Step 2: Packaging Electron Front-end ===' && \
                            npm install && \
                            npx electron-builder --win --x64
                        "
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
