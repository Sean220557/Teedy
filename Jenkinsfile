def runMaven(String args) {
    if (isUnix()) {
        sh "mvn ${args}"
    } else {
        bat "\"${env.MAVEN_CMD}\" ${args}"
    }
}

pipeline {
    agent any

    parameters {
        string(name: 'DOCKER_IMAGE', defaultValue: 'teedy-practice10', description: 'Docker image name. Use username/teedy-app when pushing to Docker Hub.')
        string(name: 'DOCKER_CREDENTIALS_ID', defaultValue: 'dockerhub_credentials', description: 'Jenkins Docker Hub credentials ID')
        booleanParam(name: 'RUN_DOCKER_STAGES', defaultValue: false, description: 'Run Practice 10 Docker build/run stages. Requires Docker Desktop/daemon to work.')
        booleanParam(name: 'PUSH_DOCKER_IMAGE', defaultValue: false, description: 'Push the image to Docker Hub. Requires DOCKER_IMAGE=username/repository and valid credentials.')
    }

    environment {
        DOCKER_TAG = "${env.BUILD_NUMBER}"
        MAVEN_CMD = 'E:\\apache-maven-3.9.14\\bin\\mvn.cmd'
        DOCKER_CMD = 'C:\\Program Files\\Docker\\Docker\\resources\\bin\\docker.exe'
    }

    stages {
        stage('Clean') {
            steps {
                script {
                    runMaven('clean')
                }
            }
        }

        stage('Compile') {
            steps {
                script {
                    runMaven('compile')
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    runMaven('test -Dmaven.test.failure.ignore=true')
                }
            }
        }

        stage('PMD') {
            steps {
                script {
                    runMaven('pmd:pmd')
                }
            }
        }

        stage('JaCoCo') {
            steps {
                script {
                    runMaven('jacoco:report')
                }
            }
        }

        stage('Javadoc') {
            steps {
                script {
                    runMaven('javadoc:javadoc')
                }
            }
        }

        stage('Site') {
            steps {
                script {
                    runMaven('site -Dmaven.test.failure.ignore=true')
                }
            }
        }

        stage('Package') {
            steps {
                script {
                    runMaven('-B -Pprod package -DskipTests')
                }
            }
        }

        stage('Build Docker Image') {
            when {
                expression { return params.RUN_DOCKER_STAGES }
            }
            steps {
                script {
                    if (isUnix()) {
                        sh "docker build -t ${params.DOCKER_IMAGE}:${env.DOCKER_TAG} -t ${params.DOCKER_IMAGE}:latest ."
                    } else {
                        bat "\"${env.DOCKER_CMD}\" build -t ${params.DOCKER_IMAGE}:${env.DOCKER_TAG} -t ${params.DOCKER_IMAGE}:latest ."
                    }
                }
            }
        }

        stage('Push Docker Image') {
            when {
                expression { return params.RUN_DOCKER_STAGES && params.PUSH_DOCKER_IMAGE }
            }
            steps {
                withCredentials([usernamePassword(credentialsId: params.DOCKER_CREDENTIALS_ID, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        if (isUnix()) {
                            sh 'printf "%s" "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                            sh "docker push ${params.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                            sh "docker push ${params.DOCKER_IMAGE}:latest"
                        } else {
                            bat "@echo %DOCKER_PASS% | \"${env.DOCKER_CMD}\" login -u \"%DOCKER_USER%\" --password-stdin"
                            bat "\"${env.DOCKER_CMD}\" push ${params.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                            bat "\"${env.DOCKER_CMD}\" push ${params.DOCKER_IMAGE}:latest"
                        }
                    }
                }
            }
        }

        stage('Run Three Containers') {
            when {
                expression { return params.RUN_DOCKER_STAGES }
            }
            steps {
                script {
                    [8082, 8083, 8084].each { port ->
                        def containerName = "teedy-container-${port}"
                        if (isUnix()) {
                            sh "docker stop ${containerName} || true"
                            sh "docker rm ${containerName} || true"
                            sh "docker run -d --name ${containerName} -p ${port}:8080 ${params.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                        } else {
                            bat "\"${env.DOCKER_CMD}\" stop ${containerName} || exit /b 0"
                            bat "\"${env.DOCKER_CMD}\" rm ${containerName} || exit /b 0"
                            bat "\"${env.DOCKER_CMD}\" run -d --name ${containerName} -p ${port}:8080 ${params.DOCKER_IMAGE}:${env.DOCKER_TAG}"
                        }
                    }

                    if (isUnix()) {
                        sh 'docker ps --filter "name=teedy-container"'
                    } else {
                        bat "\"${env.DOCKER_CMD}\" ps --filter \"name=teedy-container\""
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
