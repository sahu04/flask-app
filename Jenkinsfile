pipeline {
    agent any

    environment {
        DOCKERFILE_PATH = "./Dockerfile"
        TRIVY_REPORT_PATH = "trivy-scan-report.json"
        DOCKLE_REPORT_PATH = "dockle-scan-report.json"
    }

    stages {
        stage('Checkout') {
            steps {
                git 'https://github.com/sahu04/flask-app.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    def dockerImageName = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
                    sh "docker build -t ${dockerImageName} -f ${DOCKERFILE_PATH} ."
                    echo "Docker image name: ${dockerImageName}"
                }
            }
        }

        stage('Install Trivy') {
            steps {
                script {
                    sh 'sudo chmod o+w /usr/local/bin'
                    sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin v0.48.3'
                    sh 'wget https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl'

                    sh 'trivy --version'
                }
            }
        }

        stage('Install Dockle') {
            steps {
                 script {
                    sh 'curl -LO https://github.com/goodwithtech/dockle/releases/download/v0.4.13/dockle_0.4.13_Linux-64bit.tar.gz'
                    sh 'tar -xzf dockle_0.4.13_Linux-64bit.tar.gz'
                    sh 'sudo mv dockle /usr/local/bin/'
                    sh 'dockle --version'
                }
            }
        }

        stage('Scan image for vulnerabilities') {
            steps {
                script {
                    def dockerImageName = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
                    echo "Running Trivy scan for image: ${dockerImageName}"
                    sh "trivy  image --scanners vuln --format json -o ${TRIVY_REPORT_PATH} ${dockerImageName}"
                    sh "trivy image --scanners vuln --format template --template @./html.tpl -o report.html ${dockerImageName}"
                }
            }
        }

        stage('Security Scan - Dockle') {
            steps {
                script {
                    def dockerImageName = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
                    echo "Running Dockle scan for image: ${dockerImageName}"
                    sh "dockle -f json -o ${DOCKLE_REPORT_PATH} --exit-code 1 --exit-level fatal ${dockerImageName}"
                    
                }
            }
        }
    
        stage('Archive HTML Report') {
            steps {
                script {  
                    archiveArtifacts artifacts: 'report.html', fingerprint: true
                }
            }
        }
        
        stage('Publish HTML Report') {
            steps {
                script {
                    publishHTML(
                        target: [
                            reportDir: '.',
                            reportFiles: 'report.html',
                            reportName: 'Trivy Security Report'
                        ]
                    )
                }
            }
        }
    }
    
    post {
        always {
            script {
                archiveArtifacts artifacts: "${TRIVY_REPORT_PATH},${DOCKLE_REPORT_PATH}", followSymlinks: false
                deleteDir() // Clean the workspace
            }
        }
    }
}
