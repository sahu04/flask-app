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
        stage('Vulnerability Scan - Docker Trivy') {
            steps {
                script {
                    def dockerImageName = sh(script: "awk 'NR==1 {print \$2}' ${DOCKERFILE_PATH}", returnStdout: true).trim()
                    echo "Running Trivy scan for image: ${dockerImageName}"
                    sh "trivy --exit-code 1 --severity HIGH,MEDIUM,LOW --format json -o ${TRIVY_REPORT_PATH} ${dockerImageName}"
                }
            }
        }
    }

    post {
        always {
            script {
                // if (env.dockerImageName != "") { // Check if dockerImageName is not empty
                //     sh "docker rmi ${env.dockerImageName}" // Access dockerImageName using env prefix
                // } else {
                //     echo "Docker image name is empty. Skipping removal."
                // }
                archiveArtifacts artifacts: 'trivy-scan-report.json', followSymlinks: false
                deleteDir() // Clean the workspace
            }
        }
    }
}
