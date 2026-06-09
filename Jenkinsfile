pipeline {
    agent {
        node {
            label 'linux-slave' // Dynamically provisions your AWS instance
        }
    }

    environment {
        ARTIFACT_DIR  = 'dist'
        WINEDEBUG     = '-all'
    }

    stages {
        stage('Build Python Backend') {
            steps {
                echo '=== Step 1: Compiling FastAPI Backend via Isolated Container Pass ==='
                // This runs PyInstaller on the host workspace mount explicitly
                sh '''
                    docker run --rm \
                        -v "${WORKSPACE}":/src \
                        -w /src \
                        cdrx/pyinstaller-windows:python3 \
                        sh -c "pip install -r backend/requirements.txt && pyinstaller --onefile --windowed --name=api backend/src/api.py --distpath ./backend/bin/win"
                '''
            }
        }

        stage('Package Electron App') {
            steps {
                echo '=== Step 2: Packaging Electron Installer App via Wine ==='
                // Reclaiming permissions on the generated files so the next container can read them
                sh 'sudo chown -R $(id -u):$(id -g) "${WORKSPACE}" || true'
                
                sh '''
                    docker run --rm \
                        -v "${WORKSPACE}":/project \
                        -w /project \
                        electronuserland/builder:wine \
                        sh -c "npm install && npx electron-builder --win --x64"
                '''
            }
        }

        stage('Archive Outputs') {
            steps {
                echo '=== Step 3: Reclaiming Permissions & Archiving Final Package ==='
                sh 'sudo chown -R $(id -u):$(id -g) "${WORKSPACE}" || true'
                
                // Verification output listing
                sh 'ls -la dist/'
                
                archiveArtifacts artifacts: "dist/*.exe", allowEmptyArchive: false
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
