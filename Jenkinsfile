

pipeline{

    agent any

    environment {
    DOCKER_CREDS = credentials('dockerhubcreds')
    PROJECT_NAME= "store-management"
    DOCKER_REPO= "${DOCKER_CREDS_USR}/${PROJECT_NAME}"
    IMAGE_TAG= "${BUILD_ID}-${BRANCH_NAME}-g1s"
    STOREMANAGEMENT_DOCKER_IMAGE= "${DOCKER_REPO}:${IMAGE_TAG}"
    NODE_PORT="30080"
    SM_CONTAINER_PORT="80"
    }

    stages{

        stage('Display build data'){
            steps{
                bat 'set'
            }
        }

        stage('build jar'){
            steps{
                powershell 'mvn clean package'
            }
        }
        stage('build docker image'){
            steps{
                powershell "docker build --label GIT_COMMIT=${GIT_COMMIT} -t ${STOREMANAGEMENT_DOCKER_IMAGE} ."
                powershell "docker login -u ${DOCKER_CREDS_USR} -p ${DOCKER_CREDS_PSW}"
                powershell "docker push ${STOREMANAGEMENT_DOCKER_IMAGE}"
            }
        }
        stage('trigger deploy'){
            steps{
               withCredentials([gitUsernamePassword(credentialsId: 'githubcreds', gitToolName: 'git-tool')])  {
                powershell "git clone https://gitlab.com/storemanagement2024/infa-repo-prod.git"
                dir("infa-repo-prod"){
                    powershell "docker run -v '.:/workdir' mikefarah/yq:4 eval '.services.store-management.image = \"\"${STOREMANAGEMENT_DOCKER_IMAGE}\"\"' -i /workdir/compose.yaml"
                    powershell """
                    git add .
                    git commit -m "Upgrade version of storemanagement to ${STOREMANAGEMENT_DOCKER_IMAGE}"
                    git push
                    """
                }
                }
                
            }
        }
    }

    post{
        always {
          powershell "docker logout"
          deleteDir()
        }


    }


}