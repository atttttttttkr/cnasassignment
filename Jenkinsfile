pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'jjjxxx201'
        IMAGE_TAG = 'test'
        SERVICE = 'php-app'
        FULL_IMAGE = "${DOCKERHUB_USER}/${SERVICE}:${IMAGE_TAG}"
    }

    triggers { pollSCM('* * * * *') }

    stages {
        stage('Check Branch') {
            when {
                branch 'test'
            }
            steps {
                echo "Running on 'test' branch"
            }
        }

        stage('Checkout') {
            when {
                branch 'test'
            }
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            when {
                branch 'test'
            }
            steps {
                sh "docker build -t ${FULL_IMAGE} ${SERVICE}/"
            }
        }

        stage('Login and Push to DockerHub') {
            when {
                branch 'test'
            }
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'docker',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                    sh "docker push ${FULL_IMAGE}"
                }
            }
        }

        stage('Scan with Trivy') {
            when {
                expression { false } // This keeps the stage visible but skips execution
            }
            steps {
                script {
                    def services = ['php-app', 'mysqldatabase']
                    for (svc in services) {
                        def tag = "${env.DOCKERHUB_USER}/${svc}:${env.IMAGE_TAG}"
                        echo "Would scan ${tag} using Trivy Docker..."
                        sh """
                            docker run --rm \
                              -v /var/run/docker.sock:/var/run/docker.sock \
                              aquasec/trivy image \
                              --severity ${TRIVY_SEVERITY} \
                              --exit-code 0 \
                              --no-progress \
                              ${tag}
                        """
                    }
                }
            }
        }
        stage('Push to GitHub (Promote to master)') {
            when {
                branch 'test'
            }
            steps {
                script {
                    echo " Promoting 'test' to 'master' branch on GitHub"
                    withCredentials([usernamePassword(
                        credentialsId: 'githubcred',
                        usernameVariable: 'GIT_USER',
                        passwordVariable: 'GIT_PASS'
                    )]) {
                        sh '''
                            git config user.email "atttttttttkr@gmail.com"
                            git config user.name "atttttttttkr"
        
                            # Make sure you're on the test branch
                            git checkout test
        
                            # Push test → master (force)
                            git push https://${GIT_USER}:${GIT_PASS}@github.com/atttttttttkr/cnasassignment.git test:master --force
                        '''
                    }
                }
            }
        }
        stage('Cleanup (optional)') {
            when {
                branch 'test'
            }
            steps {
                sh """
                    docker rmi ${FULL_IMAGE} || true
                    docker system prune -f
                """
            }
        }
    }

    post {
        success {
            echo "Pipeline completed successfully"
        }
        failure {
            echo "Pipeline failed"
        }
    }
}
