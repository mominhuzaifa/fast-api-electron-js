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
                echo 'Creating workspace directory trees...'
                sh 'mkdir -p backend/bin/win'

                echo 'Writing clean deployment build script...'
                // This writes a clean build.sh script directly on the agent host workspace
                sh '''
                    cat << 'EOF' > inner-build.sh
#!/bin/bash
set -e

echo "=== Step 1: Installing Python Dependencies ==="
pip install -r backend/requirements.txt

echo "=== Step 2: Compiling Python Backend via PyInstaller ==="
pyinstaller --onefile --windowed --name=api backend/src/api.py --distpath ./backend/bin/win

echo "=== Step 3: Installing Node Modules ==="
npm install

echo "=== Step 4: Packaging Electron App ==="
npx electron-builder --win --x64
EOF
                    chmod +x inner-build.sh
                '''

                echo 'Running Build Script inside PyInstaller Wine Container...'
                // Run the script cleanly without shell quoting or heredoc bugs
                sh '''
                    docker run --rm \
                        -v "${WORKSPACE}":/src \
                        -w /src \
                        cdrx/pyinstaller-windows:python3 \
                        ./inner-build.sh
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
