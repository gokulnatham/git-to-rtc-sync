pipeline {
    options {
        skipDefaultCheckout(true)
    }
    
    environment {
        RTC_HOST = 'https://jazz.net/sandbox01-ccm/'
    }

    stages {
        stage('Preparation') {
            steps {
                script {
                    echo "Current time: ${new Date(currentBuild.startTimeInMillis)}"
                }
            }
        }

        stage('RTC Sync Code') {
             agent {
                        docker {
                        image 'na.artifactory.swg-devops.com/hyc-baw-uab-devops-modernization-team-devops-docker-local/base-images/ubuntu22-git-rtc-tools'
                        registryUrl 'https://na.artifactory.swg-devops.com'
                        registryCredentialsId '4a5bba43-7f29-474a-b1b4-78752f767f82'
                        alwaysPull true
                        reuseNode true
                        args '-u 0:0 -v /opt/app:/opt/app -v /opt/rtc-sync:/opt/rtc-sync'
                        }
                    }

            steps {
                script {
                withCredentials([usernamePassword(credentialsId: '9762a2d9-4069-414d-b077-210304c1664b', passwordVariable: 'RTC_PASSWORD', usernameVariable: 'RTC_USERNAME')]) {
                    
                          // Login to RTC_HOST inside the container
                            sh "scm login -r ${RTC_HOST} -u ${RTC_USERNAME} -P ${RTC_PASSWORD} -n local"

                          // Execute steps inside the container
                            sh "git clone https://github.com/gokulnatham/git-to-rtc-sync.git /opt/app"
                          //sh "mkdir -p /opt/rtc-sync && cd /opt/rtc-sync"
                            sh "scm load -r local --all github-sync --allow -d /opt/rtc-sync --force"
                            sh "cp -rf /opt/app/* /opt/rtc-sync/"
                            sh "scm share github-sync Standard * -r local || true"
                            sh "scm checkin /opt/rtc-sync/ --comment 'git to rtc sync' --complete"
                            sh "scm show status"
                            sh "scm deliver -v -s github-sync -r local"
                            sh "rm -rf /opt/app /opt/rtc-sync"

                          // Logout RTC_HOST inside the container
                            sh "scm logout -r ${RTC_HOST}"
                            }
                        }
                    }
                }
            }
        }