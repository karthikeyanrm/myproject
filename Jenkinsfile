pipeline {
    agent any

    environment {
        DOCKER_HUB_CREDENTIAL_ID = 'kndocker' // dockerhub credential ID
    }

    stages {
        stage('Build and Deploy') {
            steps {
                script {
                    def currentBranch = env.GIT_BRANCH
                    // Remove the "origin/" prefix if it exists
                    currentBranch = currentBranch.replaceAll("origin/", "")

                    if (currentBranch ==~ /dev/) {
                        
                        // Log in to Docker Hub securely
                        withCredentials([
                            usernamePassword(
                                credentialsId: DOCKER_HUB_CREDENTIAL_ID,
                                usernameVariable: 'DOCKER_USERNAME',
                                passwordVariable: 'MY_SECURE_PASSWORD'
                            )
                        ]) {
                            sh "echo \$MY_SECURE_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                        }

                        // Build and Deploy Docker commands for 'dev'
                        sh './build.sh'
                        sh 'docker tag reactjs-demo:latest karthikeyanrm/devrepo:latest'
                        sh 'docker push karthikeyanrm/devrepo:latest'
                        sh 'docker rmi -f reactjs-demo:latest'
                        sh 'docker rmi -f karthikeyanrm/devrepo:latest'
                        sh 'docker-compose down'
                        sh 'docker-compose -f docker-compose-dev.yml up -d'
                        
                    } else if (currentBranch ==~ /master/) {
                       
                        // Log in to Docker Hub securely
                        withCredentials([
                            usernamePassword(
                                credentialsId: DOCKER_HUB_CREDENTIAL_ID,
                                usernameVariable: 'DOCKER_USERNAME',
                                passwordVariable: 'MY_SECURE_PASSWORD'
                            )
                        ]) {
                            sh "echo \$MY_SECURE_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin"
                        }

                        // Build and Deploy Docker commands for 'master'
                        sh './build.sh'
                        sh 'docker tag reactjs-demo:latest karthikeyanrm/prodrepo:01'
                        sh 'docker push karthikeyanrm/prodrepo:01'
                        sh 'docker rmi -f reactjs-demo:latest'
                        sh 'docker rmi -f karthikeyanrm/prodrepo:01'
                        sh 'docker-compose down'
                        sh 'docker-compose -f docker-compose-prod.yml up -d'
                    } else {
                        echo "Branch not recognized: $currentBranch"
                    }
                }
            }
        }
    }

    post {
        success {
            script {
                emailext to: 'karthisk217@gmail.com',
                subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS',
                body: """The build #${BUILD_NUMBER} was successful. You can view the build log [here](${BUILD_URL}console).

Docker Container Info:
${sh(script: 'docker ps --format "{{.Names}}\\t{{.Ports}}" | awk -F "\\t" -v public_ip=$(curl -s ifconfig.me) \'{print "Container:", $1, "Public IP:", public_ip, "Port Mapping:", $2}\'', returnStdout: true)}
""",
                attachLog: true // This post success script will trigger the E-mail and Attach build log for success email
            }
        }
        failure {
            script {
                emailext to: 'karthisk217@gmail.com',
                subject: '$PROJECT_NAME - Build # $BUILD_NUMBER - $BUILD_STATUS',
                body: "The build #${BUILD_NUMBER} has failed. You can view the build log [here](${BUILD_URL}console)".toString(),
                attachLog: true // This post failure script will trigger the E-mail and Attach build log for failure email
            }
        }
    
    always {
        // Log out from Docker Hub
        sh 'docker logout'
    }
}
}
