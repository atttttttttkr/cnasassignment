pipeline {
    agent any

    environment {
        DOCKERHUB_USER = 'jjjxxx201'
        IMAGE_TAG = 'test2'
        TRIVY_SEVERITY = 'CRITICAL,HIGH'
    }

    stages {
        stage('Check Branch') {
            when {
                expression { env.BRANCH_NAME == 'test2' }
            }
            steps {
                echo "✅ Running on 'test2' branch"
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

        // /* stage('Scan with Trivy') {
        //     steps {
        //         script {
        //             def services = ['php-app', 'mysqldatabase']
        //             for (svc in services) {
        //                 def tag = "${env.DOCKERHUB_USER}/${svc}:${env.IMAGE_TAG}"
        //                 echo "🔍 Scanning ${tag} using Trivy Docker"
        //                 sh """
        //                     docker run --rm \
        //                       -v /var/run/docker.sock:/var/run/docker.sock \
        //                       aquasec/trivy image \
        //                       --severity ${TRIVY_SEVERITY} \
        //                       --exit-code 0 \
        //                       --no-progress \
        //                       ${tag}
        //                 """
        //             }
        //         }
        //     }
        // } */

        stage('Push Test Images') {
            when {
                expression { return true }
            }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
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

        stage('Promote to Latest') {
            when {
                expression { return true }
            }
            steps {
                script {
                    withCredentials([usernamePassword(
                        credentialsId: 'docker',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )]) {
                        sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
                        def services = ['php-app', 'mysqldatabase']
                        for (svc in services) {
                            def testTag = "${env.DOCKERHUB_USER}/${svc}:${env.IMAGE_TAG}"
                            def latestTag = "${env.DOCKERHUB_USER}/${svc}:latest"
                            sh """
                                docker pull ${testTag}
                                docker tag ${testTag} ${latestTag}
                                docker push ${latestTag}
                            """
                        }
                    }
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    def services = ['php-app', 'mysqldatabase']
                    for (svc in services) {
                        def testTag = "${env.DOCKERHUB_USER}/${svc}:${env.IMAGE_TAG}"
                        def latestTag = "${env.DOCKERHUB_USER}/${svc}:latest"
                        sh """
                            docker rmi ${testTag} || true
                            docker rmi ${latestTag} || true
                            docker system prune -f
                        """
                    }
                }
            }
        }
        stage('Deploy to master') {
            when {
                branch 'test2'
            }
            steps {
                script {
                    echo "🚀 Deploying to master (without Jenkinsfile)"
                    withCredentials([usernamePassword(credentialsId: 'githubcred', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASS')]) {
                        sh '''
                            git config user.email "atttttttttkr@gmail.com"
                            git config user.name "atttttttttkr"
                            git init
                            git add .
                            git commit -m "Passed Jenkins Test"
                            git remote add origin https://github.com/atttttttttkr/cnasassignment.git
                            git push https://${GIT_USER}:${GIT_PASS}@github.com/atttttttttkr/cnasassignment.git master --force

                        '''
                    }
                }
            }
        }
    }
    post {
        failure {
            echo "❌ Pipeline failed. No merge."
        }
        success {
            echo "✅ All stages passed. Job done."
        }
        always {
            cleanWs()
        }
    }
}
