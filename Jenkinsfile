pipeline {
    agent any
    
    environment {
        DOCKER_REPO_HOST = "na.artifactory.swg-devops.com"
        RTC_HOST = 'https://jazz.net/sandbox01-ccm/'
        DOCKER_IMAGE = "${DOCKER_REPO_HOST}/hyc-baw-uab-devops-modernization-team-devops-docker-local/base-images/ubuntu22-scm-git-workitem"
    }

    stages {
        stage('Preparation') {
            steps {
                script {
                    echo "Current time: ${new Date(currentBuild.startTimeInMillis)}"
                    
                    // Check and stop/remove existing container
                    sh 'docker stop rtc-sync || true'
                    sh 'docker rm rtc-sync || true'
                    
                    // Login to JFrog Artifactory
                    withCredentials([usernamePassword(credentialsId: '4a5bba43-7f29-474a-b1b4-78752f767f82', passwordVariable: 'DOCKER_PASSWORD', usernameVariable: 'DOCKER_USERNAME')]) {
                        sh "docker login -u ${DOCKER_USERNAME} -p ${DOCKER_PASSWORD} ${DOCKER_REPO_HOST}"
                    }

                    // Pull Docker image
                    sh "docker pull ${DOCKER_IMAGE}"
                }
            }
        }

        stage('RTC Sync Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: '9762a2d9-4069-414d-b077-210304c1664b', passwordVariable: 'RTC_PASSWORD', usernameVariable: 'RTC_USERNAME')]) {
                    // Use docker run to start a container
                    script {
                        sh "docker run -d -i --name rtc-sync -v /var/run/docker.sock:/var/run/docker.sock ${DOCKER_IMAGE}"

                        // Wait for the container to start
                        sleep 10

                        // Login to RTC_HOST inside the container
                        sh "docker exec rtc-sync scm login -r ${RTC_HOST} -u ${RTC_USERNAME} -P ${RTC_PASSWORD} -n local"
                        sh """docker exec rtc-sync sh -c 'echo "${RTC_PASSWORD}" | ewmcli.py login -r "${RTC_HOST}" -u "${RTC_USERNAME}" -n local -p "BAW Project"'"""

                        // Execute steps inside the container
                        sh "docker exec rtc-sync sh -c '/root/workitemsync.sh'"

                        // Logout RTC_HOST inside the container
                        sh "docker exec rtc-sync scm logout -r ${RTC_HOST}"
                        sh "docker exec rtc-sync ewmcli.py logout -r ${RTC_HOST}"

                        // Stop and remove the container
                        // sh "docker stop rtc-sync"
                        // sh "docker rm rtc-sync"
                    }
                }
            }
        }
    }
}
