pipeline {
    agent { label 'jenkins_node' }

    environment {
        APP_PORT = "4000"
      
        DOCKER_REPO = "emman159/product_frontend"
    }

    stages {

        stage('Build & Deploy') {
            steps {
                script {

                    def BRANCH = env.BRANCH_NAME
                    def EXT_PORT = 9090
                    def VERSION_TAG = "${BRANCH}-${BUILD_NUMBER}" // second tag

                    echo "Building for branch: ${BRANCH}"

                    if (BRANCH == 'main') {
                        EXT_PORT = 4001
                    } else if (BRANCH == 'development') {
                        EXT_PORT = 7076
                    } else if (BRANCH == 'staging') {
                        EXT_PORT = 7077
                    } else {
                        EXT_PORT = 8089
                    }

                    sh """
                        echo 'EXT_PORT=${EXT_PORT}' > .env
                        echo 'BRANCH=${BRANCH}' >> .env
                        echo 'DOCKER_REPO=${DOCKER_REPO}' >> .env
                    """

                    // Build Docker image with 2 tags
                    sh """
                        docker build -t ${DOCKER_REPO}:${BRANCH}-latest \
                                     -t ${DOCKER_REPO}:${VERSION_TAG} .
                    """

                    // Docker Hub login
                    withCredentials([usernamePassword(credentialsId: 'dockerhub-auth', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                        sh "echo $PASS | docker login -u $USER --password-stdin"
                    }

                    // Push both tags
                    sh """
                        docker push ${DOCKER_REPO}:${BRANCH}-latest
                        docker push ${DOCKER_REPO}:${VERSION_TAG}
                    """

                    // Deploy
                    sh """
                        docker compose --env-file .env up -d --force-recreate
                    """
                }
            }
        }
    }
}
