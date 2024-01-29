// pipeline {
//     agent any

//     environment {
//         DOCKERFILE_PATH = "./Dockerfile"
//         DOCKER_IMAGE_NAME = ""
//         TRIVY_REPORT_PATH = "trivy-scan-report.json"
//     }

//     stages {
//         stage('Checkout') {
//             steps {
//                 git 'https://github.com/sahu04/flask-app.git'
//             }
//         }

//         stage('Build Docker Image') {
//             steps {
//                 script {
//                     // Set dockerImageName as an environment variable
//                     DOCKER_IMAGE_NAME = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
//                     sh "docker build -t ${DOCKER_IMAGE_NAME} -f ${DOCKERFILE_PATH} ."
//                 }
//             }
//         }

//         stage('Install Trivy') {
//             steps {
//                 script {
//                     sh '''
//                         sudo wget -qO trivy https://github.com/aquasecurity/trivy/releases/latest/download/trivy_$(uname -s)_$(uname -m)
//                         sudo chmod +x trivy
//                         sudo mv trivy /usr/local/bin/
//                     '''
//                     sh 'trivy --version'
//                 }
//             }
//         }

//         stage('Vulnerability Scan - Docker Trivy') {
//             steps {
//                 script {
//                     echo "Running Trivy scan for image: ${DOCKER_IMAGE_NAME}"
//                     sh "trivy --exit-code 1 --severity HIGH,MEDIUM,LOW --format json -o ${TRIVY_REPORT_PATH} ${DOCKER_IMAGE_NAME}"

//                 }
//             }
//         }
//     }

//     post {
//         always {
//             script {
//                 sh "docker rmi ${DOCKER_IMAGE_NAME}"
//                 archiveArtifacts artifacts: 'trivy-scan-report.json', followSymlinks: false
//             }
//         }
//     }
// }

pipeline {
    agent any

    environment {
        DOCKERFILE_PATH = "./Dockerfile"
        TRIVY_REPORT_PATH = "trivy-scan-report.json"
        dockerImageName = "" // Define dockerImageName globally
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
                    
                    // Continue using dockerImageName directly in the subsequent steps
                    echo "Docker image name: ${dockerImageName}"
                }
            }
        }

        stage('Install Trivy') {
            steps {
                script {
                    sh 'sudo chmod o+w /usr/local/bin'
                    sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/install.sh | sh -s -- -b /usr/local/bin v0.18.3'
                    sh 'curl -sfL https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl > html.tpl'

                    sh 'trivy --version'
                }
            }
        }
        stage('Install Dockle') {
            steps {
                 script {
                    // Download Dockle tarball
                    sh 'curl -LO https://github.com/goodwithtech/dockle/releases/download/v0.4.13/dockle_0.4.13_Linux-64bit.tar.gz'

                    // Extract Dockle tarball
                    sh 'tar -xzf dockle_0.4.13_Linux-64bit.tar.gz'

                    // Move Dockle binary to a directory in the system path
                    sh 'sudo mv dockle /usr/local/bin/'

                    // Verify Dockle installation
                    sh 'dockle --version'
                }
            }
        }
        stage('Vulnerability Scan - Docker Trivy') {
            steps {
                script {
                    def dockerImageName = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
                    echo "Running Trivy scan for image: ${dockerImageName}"
                    // sh "trivy --exit-code 1 --severity HIGH,MEDIUM,LOW --format json -o ${TRIVY_REPORT_PATH} ${dockerImageName}"
                    sh "trivy  --severity HIGH,MEDIUM,LOW --format json -o ${TRIVY_REPORT_PATH} ${dockerImageName}"
                }
            }
        }
        stage('Security Scan - Dockle') {
            steps {
                script {
                    def dockerImageName = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
                    echo "Running Dockle scan for image: ${dockerImageName}"
                     sh "dockle --exit-code 1 --exit-level fatal -o json -f ${DOCKLE_REPORT_PATH} ${dockerImageName}"
                     // sh "dockle  --exit-level fatal -o json -f ${DOCKLE_REPORT_PATH} ${dockerImageName}"
                }
            }
        }
    } 

    post {
    always {
        script {
            archiveArtifacts artifacts: "${TRIVY_REPORT_PATH}, ${DOCKLE_REPORT_PATH}", followSymlinks: false
            deleteDir() // Clean the workspace
            }
        }
    }
} 
