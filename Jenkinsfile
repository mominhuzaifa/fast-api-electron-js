pipeline {
    agent none 

    environment {
        ARTIFACT_DIR  = 'dist'
        PYTHON_DIST   = 'backend/dist'
        WINEDEBUG     = '-all'
        SONAR_URL     = "${env.SONAR_HOST_URL}"
    }

    stages {
        stage('Linting') {
            agent {
                docker { 
                    image 'node:20-alpine' 
                    reuseNode true
                    args '-e npm_config_cache=/tmp/.npm'
                }
            }
            steps {
                echo 'Running Frontend Linting...'
                sh '''
                    npm install
                    npm run lint --if-present
                '''
            }
        }

        stage('SonarQube Static Analysis') {
            agent any
            steps {
                echo 'Running SonarQube Analysis...'
                withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                    sh '''
                        docker run --rm \
                            -v "${WORKSPACE}:/usr/src" \
                            -v "sonar_cache:/opt/sonar-scanner/.sonar/cache" \
                            sonarsource/sonar-scanner-cli:latest \
                            -Dsonar.projectKey=fast-api-electron-js \
                            -Dsonar.projectName="FastAPI Electron Windows App" \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_URL} \
                            -Dsonar.token=${SONAR_TOKEN}
                    '''
                }
            }
        }

        stage('Build Windows App inside Wine-Docker') {
            agent {
                docker {
                    image 'electronuserland/builder:wine-chrome'
                    reuseNode true
                }
            }
            steps {
                echo 'Compiling Python FastAPI Backend to Windows EXE...'
                sh '''
                    wine pip install -r backend/requirements.txt
                    wine pyinstaller --onefile --windowed --name=api backend/src/api.py --distpath ./${PYTHON_DIST}
                '''

                echo 'Placing Python Binary into Electron Assets...'
                sh '''
                    mkdir -p backend/bin/win
                    cp ${PYTHON_DIST}/api.exe backend/bin/win/
                '''

                echo 'Building Final Windows Installer via Electron Builder...'
                sh '''
                    npm install
                    npx electron-builder --win --x64
                '''
            }
        }

        stage('Archive Windows Installer') {
            agent any
            steps {
                echo 'Archiving build artifacts...'
                archiveArtifacts artifacts: "${ARTIFACT_DIR}/*.exe", allowEmptyArchive: false
            }
        }
    }

    // FIX: Explicit declarative agent definition inside the post block
    post {
        always {
            node('built-in') {
                echo 'Cleaning up workspace execution contexts...'
                cleanWs()
            }
        }
    }
}
