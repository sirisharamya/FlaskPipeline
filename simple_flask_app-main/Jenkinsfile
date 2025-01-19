pipeline {
    agent any 
    
    environment {
        VENV_PATH = 'venv'
        PORT = '8000'
    }
    
    stages {
        stage('Clone Repository') {
            steps {
                cleanWs()
                git branch: 'main',
                    url: 'https://github.com/sirisharamya/FlaskPipeline.git'
            }
        }
        
        stage('Set Up Python Environment') {
            steps {
                script {
                    // Create a virtual environment and upgrade pip
                    bat 'python -m venv venv'
                    bat 'venv\\Scripts\\activate.bat && python -m pip install --upgrade pip'
                    
                    // Install dependencies from requirements.txt inside the nested folder
                    bat '''
                        venv\\Scripts\\activate.bat && (
                            pip install -r simple_flask_app-main/requirements.txt --verbose
                        )
                    '''
                }
            }
        }
        
        stage('Run Tests') {
            steps {
                script {
                    // Run pytest inside the nested folder
                    bat 'venv\\Scripts\\activate.bat && cd simple_flask_app-main && python -m pytest'
                }
            }
        }
        
        stage('Deploy Application') {
            steps {
                script {
                    // Create a Windows batch script to run the Flask server
                    bat '''
                        echo @echo off > start_server.bat
                        echo set FLASK_APP=simple_flask_app-main/app.py >> start_server.bat
                        echo set FLASK_ENV=production >> start_server.bat
                        echo call venv\\Scripts\\activate.bat >> start_server.bat
                        echo python -m flask run --host=127.0.0.1 --port=%PORT% >> start_server.bat
                    '''
                    
                    // Start the server in the background
                    bat 'start /B start_server.bat'
                    
                    // Wait for the server to start
                    bat '''
                        powershell -Command "Start-Sleep -Seconds 15"
                    '''
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    // Verify that the application is running
                    bat '''
                        powershell -Command "try { \
                            \$response = Invoke-WebRequest -Uri http://127.0.0.1:%PORT% -UseBasicParsing; \
                            if (\$response.StatusCode -eq 200) { \
                                Write-Host 'Application is running successfully!'; \
                                exit 0; \
                            } else { \
                                Write-Host 'Application returned status code: ' \$response.StatusCode; \
                                exit 1; \
                            } \
                        } catch { \
                            Write-Host 'Failed to connect to application: ' \$_.Exception.Message; \
                            exit 1; \
                        }"
                    '''
                }
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    // Perform a health check
                    bat '''
                        powershell -Command "try { \
                            \$response = Invoke-WebRequest -Uri http://127.0.0.1:%PORT%/health -UseBasicParsing; \
                            \$content = \$response.Content | ConvertFrom-Json; \
                            if (\$content.status -eq 'healthy') { \
                                Write-Host 'Health check passed!'; \
                                exit 0; \
                            } else { \
                                Write-Host 'Health check failed: ' \$content.status; \
                                exit 1; \
                            } \
                        } catch { \
                            Write-Host 'Health check failed: ' \$_.Exception.Message; \
                            exit 1; \
                        }"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            script {
                // Clean up running processes
                bat '''
                    powershell -Command "try { \
                        Get-Process -Name python -ErrorAction SilentlyContinue | Stop-Process -Force -ErrorAction SilentlyContinue; \
                        Get-Process -Name flask -ErrorAction SilentlyContinue | Stop-Process -Force -ErrorAction SilentlyContinue; \
                        exit 0; \
                    } catch { \
                        exit 0; \
                    }"
                '''
            }
        }
        failure {
            script {
                // Log installed packages on failure
                echo 'Pipeline failed! Checking virtual environment status...'
                bat '''
                    echo Listing installed packages:
                    venv\\Scripts\\activate.bat && pip list
                '''
            }
        }
    }
}
