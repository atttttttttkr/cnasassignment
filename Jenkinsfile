pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'jjjxxx201'
        IMAGE_TAG = 'test' // Can be BUILD_NUMBER or commit SHA
        TRIVY_SEVERITY = 'CRITICAL,HIGH'
    }

    stages {

        stage('Check Branch') {
            when {
                expression { env.BRANCH_NAME == 'test' }
            }
            steps {
                echo "✅ Running on 'test' branch"
            }
        }

        stage('Build Docker Images') {
            steps {
                script {
                    def services = ['php-app', 'mysqldatabase']
                    for (svc in services) {
                        def tag = "${env.DOCKERHUB_USER}/${svc}:${env.IMAGE_TAG}"
                        echo "🔨 Building ${tag}"
                        sh "docker build -t ${tag} ${svc}/"
                    }
                }
            }
        }

        stage('Scan with Trivy') {
            steps {
                script {
                    def services = ['php-app', 'mysql']
                    for (svc in services) {
                        def tag = "${env.DOCKERHUB_USER}/${svc}:${env.IMAGE_TAG}"
                        echo "🔍 Scanning ${tag} using Trivy Docker"
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

        stage('Push to Docker Hub (optional)') {
            when {
                expression { return true } // set to true to enable pushing
            }
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        def services = ['php-app', 'mysqldatabase']
                        for (svc in services) {
                            def tag = "${env.DOCKERHUB_USER}/${svc}:${env.IMAGE_TAG}"
                            sh "docker push ${tag}"
                        }
                    }
                }
            }
        }

        stage('Merge to main') {
            when {
                branch 'test2'
            }
            steps {
                script {
                    echo "🛠️ Merging test -> main using GitHub credentials"
                    withCredentials([usernamePassword(credentialsId: 'githubcred', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        sh '''
                            git config user.email "atttttttttkr@gmail.com"
                            git config user.name "atttttttttkr"
                            git checkout main || git checkout -b main
                            git merge test -m "Auto-merge from test after passing CI"
                            git push https://${GIT_USER}:${GIT_PASS}@github.com/atttttttttkr/cnasassignment.git main
                        '''
                    }
                }
            }
        }
    } // <--- CLOSES "stages"

    post {
        failure {
            echo "❌ Pipeline failed. No merge."
        }
        success {
            echo "✅ All stages passed. Job done."
        }
    }
} // <--- CLOSES "pipeline"
