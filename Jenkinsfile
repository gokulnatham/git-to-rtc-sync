pipeline {
    agent any
    
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
            steps {
                withCredentials([usernamePassword(credentialsId: '9762a2d9-4069-414d-b077-210304c1664b', passwordVariable: 'RTC_PASSWORD', usernameVariable: 'RTC_USERNAME')]) {
                    // Use docker run to start a container
                    script {
                        sh 'sudo scm login -u ${RTC_USERNAME} -P ${RTC_PASSWORD} -r ${RTC_HOST} -n local'
                        sh 'sudo ewmcli.py create Task -t /root/ewmcli/tasktemplate.json -r local -p "BAW Project" > ${WORKSPACE}/itemnumber.txt'    
                        sh "sudo git clone https://github.com/gokulnatham/git-to-rtc-sync.git /opt/app"
                        sh "sudo mkdir -p /opt/rtc-sync && cd /opt/rtc-sync"
                        sh "sudo scm load -r local --all github-sync --allow -f"
                        sh "sudo cp -rf /opt/app/* /opt/rtc-sync/ && sudo rm -rf /opt/app/"
                        sh "sudo scm share github-sync Standard * -r local || true"
                        sh "sudo scm checkin . --comment \"git to rtc sync\" --complete"
                        sh 'sudo scm show status > ${WORKSPACE}/changeset.txt'
                        sh "sudo scm deliver -s github-sync -r local"

                        // Logout RTC_HOST inside the container
                        sh "sudo scm logout -r ${RTC_HOST}"

                    }
                }
            }
        }
    }
}
