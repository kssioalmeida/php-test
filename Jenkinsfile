stage 'Checkout'
 node() {
  deleteDir()
  checkout scm
}
stage 'Slack Notification'
 node () {
  sh 'git log -1 --pretty=%B > commit-log.txt'
  GIT_COMMIT=readFile('commit-log.txt').trim()
  slackSend channel: 'integrationtests', color: '#1e602f', message: ":octocat: - BUILD_STARTED: PROJECT - ${env.JOB_NAME} - (${GIT_COMMIT})"
}
stage 'DEV Build'
 node () {
  openshiftBuild(buildConfig: 'phpdev', showBuildLogs: 'true')
}
stage 'DEV Check'
 node () {
  openshiftVerifyBuild(buildConfig: 'phpdev')
}
pipeline {
  agent none
  stages {
    stage("Distributed Tests") {
      steps {
        parallel (
          "Firefox" : {
            node('master') {
              sh "echo from Firefox"
            }
          },
          "Chrome" : {
            node('master') {
              sh "echo from Chrome"
            }
          },
          "IE6 : )" : {
            node('master') {
              sh "echo from IE6"
            }
          }
        )
      }
    }
  }
}
stage 'Promote to QA'
 node () {
   openshiftTag(srcStream: "phpdev", srcTag: "latest", destStream: "phpdev", destTag: "qaready")
}
stage 'QA Check'
 node () {
  openshiftVerifyDeployment(deploymentConfig: 'phpqa')
}
stage 'Approval' {
  slackSend channel: 'integrationtests', color: '#42e2f4', message: ":dusty_stick: - CTO - Please evaluate the Project - ${env.JOB_NAME} - http://jenkins-meu-teste.getup.io/blue/organizations/jenkins/${env.JOB_NAME}/detail/${env.JOB_NAME}/${env.BUILD_NUMBER}/pipeline/ "
  try {
    input message: 'Are this version ready for Production ?', submitter: 'juniorjbn'
  } catch(err) {
    slackSend channel: 'integrationtests', color: '#d80f41', message: ":finnadie: - Build (${env.BUILD_NUMBER}) from Project - ${env.JOB_NAME} - ABORTED in QA "
  }
}
stage 'Promote to PROD'
 node () {
   openshiftTag(srcStream: "phpdev", srcTag: "qaready", destStream: "phpdev", destTag: "prodready")
}
stage 'PROD Check'
 node () {
  openshiftVerifyDeployment(deploymentConfig: 'phpprod')
}
stage 'Slack Notification'
  node () {
   sh 'git log -1 --pretty=%B > commit-log.txt'
   GIT_COMMIT=readFile('commit-log.txt').trim()
   slackSend channel: 'integrationtests', color: '#1e602f', message: ":thumbsup_all: - UPDATE approved to production: PROJECT - ${env.JOB_NAME} - Build Number - ${env.BUILD_NUMBER} - (${GIT_COMMIT})"
}
