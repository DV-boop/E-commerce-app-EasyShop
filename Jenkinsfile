@Library("jenkinsLibrary") _

pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'dvharsh/easyshop'
        DOCKER_MIGRATION_IMAGE = 'dvharsh/easyshop-migration'
        DOCKER_CREDENTIALS = "dockerHubCredentials"
        EMAIL_ADDRESS = "dvharsh9@gmail.com"

        // ✅ Add these two:
        SONAR_SCANNER_HOME = "/opt/sonar-scanner"
        PATH = "${SONAR_SCANNER_HOME}/bin:${PATH}"
    }

    stages {
        stage("Set Build Tags") {
            steps {
                script {
                    env.DOCKER_TAG = "${BUILD_NUMBER}"
                }
            }
        }

        stage("Clean Workspace") {
            steps {
                cleanWorkspace()
            }
        }

        stage("Code Repository") {
            steps {
                cloneRepository(
                    branch: "hackathon",
                    repoUrl: "https://github.com/DV-boop/E-commerce-app-EasyShop.git"
                )
            }
        }

        stage("Trivy File System Scanning") {
            steps {
                trivyFileSystemScan()
            }
        }

        stage("SonarQube Quality Analysis") {
            steps {
                sonarQubeAnalysis(
                    sonarQubeTokenName: 'sonarQubeToken', 
                    sonarQubeProjectKey: 'ecom', 
                    sonarQubeProjectName: 'e-commerce', 
                    sonarQubeInstallationName: 'sonarQubeScanner'
                )
            }
        }

        stage("Docker Image Build") {
            parallel {
                stage("Build Main Docker Image") {
                    steps {
                        dockerBuild(
                            imageName: env.DOCKER_IMAGE,
                            imageTag: env.DOCKER_TAG
                        )
                    }
                }
                stage("Build Migration Docker Image") {
                    steps {
                        dockerBuild(
                            imageName: env.DOCKER_MIGRATION_IMAGE,
                            imageTag: env.DOCKER_TAG,
                            dockerfile: './scripts/Dockerfile.migration',
                            context: '.'
                        )
                    }
                }
            }
        }

        stage("Trivy Image Scanning") {
            steps {
                trivyImageScan(
                    imageName: env.DOCKER_IMAGE, 
                    imageTag: env.DOCKER_TAG
                )
            }
        }

        stage("Push Docker Image") {
            parallel {
                stage("Pushing Main Docker Image") {
                    steps {
                        dockerPush(
                            imageName: env.DOCKER_IMAGE,
                            imageTag: env.DOCKER_TAG,
                            credentialsId: env.DOCKER_CREDENTIALS
                        )
                    }
                }
                stage("Pushing Migration Docker Image") {
                    steps {
                        dockerPush(
                            imageName: env.DOCKER_MIGRATION_IMAGE,
                            imageTag: env.DOCKER_TAG,
                            credentialsId: env.DOCKER_CREDENTIALS
                        )
                    }
                }
            }
        }
    }

    post {
        always {
            emailNotification(env.EMAIL_ADDRESS, ['trivy-image-report.txt', 'trivy-fs-report.txt'])
        }
    }
}
