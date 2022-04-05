#!groovy

pipeline {
    agent none
    stages {
        stage('Build') {

            agent any
            tools {
                maven 'M3'
            }
            steps {
                sh 'mvn clean install'
            }
        }
        stage('Tests') {
            agent any
            tools {
                maven 'M3'
            }
            steps {
                sh 'mvn test'
            }
        }
        stage('Gerar versão') {
            agent any
            tools {
                maven 'M3'
            }
            steps {
                script {
                    if (env.BRANCH_NAME == 'master') {
                        echo 'Master'
                        PRE_RELEASE = ''
                        TAG = VersionNumber(versionNumberString: '${BUILD_DATE_FORMATTED, "yyyyMMdd"}.${BUILDS_TODAY}.${BUILD_NUMBER}')
                    } else {
                        echo 'Dev'
                        PRE_RELEASE = ' --pre-release'
                        TAG = 'Alpha-' + VersionNumber(versionNumberString: '${BUILD_DATE_FORMATTED, "yyyyMMdd"}.${BUILDS_TODAY}.${BUILD_NUMBER}')
                    }
                }
                sh 'git pull origin master'
                sh 'mvn versions:set versions:commit -DnewVersion=' + TAG
                sh 'mvn clean install'

                echo "Compressing artifacts into one file"
                sh 'zip -r notify-sync-client.zip target'

                withCredentials([usernamePassword(credentialsId: 'github_global', passwordVariable: 'password', usernameVariable: 'user')]) {
                    echo "Creating a new release in github"
                    sh 'github-release release --user krlsedu --security-token ' + env.password + ' --repo notify-sync-client --tag ' + TAG + ' --name "' + TAG + '"' + PRE_RELEASE

                    echo "Uploading the artifacts into github"
                    sleep(time: 3, unit: "SECONDS")

                    sh 'github-release upload --user krlsedu --security-token ' + env.password + ' --repo notify-sync-client --tag ' + TAG + ' --name notify-sync-client"' + TAG + '.zip" --file notify-sync-client.zip'

                    script {
                        if (env.BRANCH_NAME == 'master') {
                            sh "git add ."
                            sh "git config --global user.email 'krlsedu@gmail.com'"
                            sh "git config --global user.name 'Carlos Eduardo Duarte Schwalm'"
                            sh "git commit -m 'Triggered Build: " + TAG + "'"
                            sh 'git push https://krlsedu:${password}@github.com/krlsedu/notify-sync-client.git HEAD:' + env.BRANCH_NAME
                        }
                    }
                }
            }
        }
    }
}