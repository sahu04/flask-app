pipeline {
    agent any

    stages {
        stage('Checkout') {
            steps {
                script {
                    // Checkout your Git repository
                    checkout scm
                }
            }
        }

        stage('Install Dependencies') {
            steps {
                script {
                    // Install required dependencies
                    sh 'pip install --upgrade pip'
                    sh 'pip install -r requirements.txt'
                }
            }
        }

        stage('Deploy to Local Server') {
            steps {
                script {
                    // Your deployment commands
                    sh 'nohup python3 main.py &'
                    sleep 10  // Give some time for the app to start
                }
            }
        }

        stage('Check Application URL') {
            steps {
                script {
                    // Use curl to check if the application is responsive
                    def responseCode = sh(script: 'curl -o /dev/null -s -w "%{http_code}" http://localhost:5000', returnStatus: true)
                    
                    if (responseCode == 200) {
                        // Print the URL if the application is accessible
                        echo 'Application is accessible.'
                        echo 'URL: http://localhost:5000'
                    } else {
                        error "Application health check failed. HTTP status code: ${responseCode}"
                    }
                }
            }
        }
    }
}

