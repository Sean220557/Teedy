pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_IMAGE', defaultValue: 'your-dockerhub-username/teedy-app', description: 'Docker Hub repository, for example username/teedy-app')
        string(name: 'DOCKER_CREDENTIALS_ID', defaultValue: 'dockerhub_credentials', description: 'Jenkins Docker Hub credentials ID')
    }

    environment {
        DOCKER_TAG = "${env.BUILD_NUMBER}"
    }

    stages {
        stage('Clean') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn clean'
                    } else {
                        bat 'mvn clean'
                    }
                }
            }
        }

        stage('Compile') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn compile'
                    } else {
                        bat 'mvn compile'
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn test -Dmaven.test.failure.ignore=true'
                    } else {
                        bat 'mvn test -Dmaven.test.failure.ignore=true'
                    }
                }
            }
        }

        stage('PMD') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn pmd:pmd'
                    } else {
                        bat 'mvn pmd:pmd'
                    }
                }
            }
        }

        stage('JaCoCo') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn jacoco:report'
                    } else {
                        bat 'mvn jacoco:report'
                    }
                }
            }
        }

        stage('Javadoc') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn javadoc:javadoc'
                    } else {
                        bat 'mvn javadoc:javadoc'
                    }
                }
            }
        }

        stage('Site') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn site -Dmaven.test.failure.ignore=true'
                    } else {
                        bat 'mvn site -Dmaven.test.failure.ignore=true'
                    }
                }
            }
        }

        stage('Package') {
            steps {
                script {
                    if (isUnix()) {
                        sh 'mvn -B -Pprod package -DskipTests'
                    } else {
                        bat 'mvn -B -Pprod package -DskipTests'
                    }
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    if (isUnix()) {
                        sh "docker build -t ${params.DOCKER_IMAGE}:${env.DOCKER_TAG} -t ${params.DOCKER_IMAGE}:latest ."
                    } else {
                        bat "docker build -t ${params.DOCKER_IMAGE}:${env.DOCKER_TAG} -t ${params.DOCKER_IMAGE}:latest ."
                    }
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: params.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        if (isUnix()) {
                            sh 'printf "%s" "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                            sh "docker push ${params.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                            sh "docker push ${params.DOCKER_IMAGE}:latest"
                        } else {
                            bat '@echo %DOCKER_PASS% | docker login -u "%DOCKER_USER%" --password-stdin'
                            bat "docker push ${params.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                            bat "docker push ${params.DOCKER_IMAGE}:latest"
                        }
                    }
                }
            }
        }

        stage('Run Three Containers') {
            steps {
                script {
                    [8082, 8083, 8084].each { port ->
                        def containerName = "teedy-container-${port}"
                        if (isUnix()) {
                            sh "docker stop ${containerName} || true"
                            sh "docker rm ${containerName} || true"
                            sh "docker run -d --name ${containerName} -p ${port}:8080 ${params.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                        } else {
                            bat "docker stop ${containerName} || exit /b 0"
                            bat "docker rm ${containerName} || exit /b 0"
                            bat "docker run -d --name ${containerName} -p ${port}:8080 ${params.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                        }
                    }

                    if (isUnix()) {
                        sh 'docker ps --filter "name=teedy-container"'
                    } else {
                        bat 'docker ps --filter "name=teedy-container"'
                    }
                }
            }
        }
    }

    post {
        always {
            junit allowEmptyResults: true, testResults: '**/target/surefire-reports/*.xml'
            archiveArtifacts artifacts: '**/target/site/**/*.*', allowEmptyArchive: true, fingerprint: true
            archiveArtifacts artifacts: '**/target/**/*.jar', allowEmptyArchive: true, fingerprint: true
            archiveArtifacts artifacts: '**/target/**/*.war', allowEmptyArchive: true, fingerprint: true
        }
    }
}
