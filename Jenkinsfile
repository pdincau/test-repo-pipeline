node {
  def projectPath = env.JOB_NAME.tokenize( '/' )
  def organization = projectPath[0]
  def projectName = projectPath[1]

  env.RTC_URL="https://10.0.0.112:9443/ccm"
  env.RTC_USERNAME="valentina"
  env.RTC_PASSWORD="valentina"
  env.REMOTE_WORKSPACE="sync-workspace$BUILD_NUMBER"
  env.LOCAL_WORKSPACE="local-sync-workspace"
  env.GIT_CLONE_DIRECTORY="git-repo"

  try {
    stage('Git clone') {
        dir('git-repo') {
          checkout([$class: 'GitSCM', branches: [[name: '*/*']], userRemoteConfigs: [[url: "https://github.com/${organization}/${projectName}.git"]]])
          env.BRANCH_NAME = branchName()
       }
     }
     stage('RTC clone') {
        scm 'login -u $RTC_USERNAME -P $RTC_PASSWORD -r $RTC_URL'
        scm 'create workspace -r $RTC_URL -s $BRANCH_NAME $REMOTE_WORKSPACE'
        scm 'load -r $RTC_URL -f -d $LOCAL_WORKSPACE --all $REMOTE_WORKSPACE'
     }
     stage('Sync Git to RTC') {
        sync '$GIT_CLONE_DIRECTORY', "${env.LOCAL_WORKSPACE}/SRC/$projectName"
        scm 'checkin --comment "synch commit" $LOCAL_WORKSPACE'
        scm 'deliver -d $LOCAL_WORKSPACE --source $REMOTE_WORKSPACE -r $RTC_URL'
    }
  }
  catch(exception) {
    mailSyncFailed()
    throw exception
  }
  finally {
    scm 'delete workspace -r $RTC_URL $REMOTE_WORKSPACE'
  }
}

def scm(command) {
  sh "/opt/scmtools/eclipse/scm $command"
}

def sync(gitFolder, rtcFolder) {
  sh "rsync -av --progress $gitFolder/* $rtcFolder --exclude .git --delete"
}

def mailSyncFailed() {
   mail subject: "Sync to RTC failed in Jenkins: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
        to: 'paolo.dincau@xpeppers.com',
        body: "Sync to RTC ${env.JOB_NAME} failed in Jenkins.\n\nSee ${env.BUILD_URL}"
}

def branchName() {
  def completeName = sh(script: 'git name-rev --name-only HEAD', returnStdout: true)
  def matcher = completeName =~ /remotes\/origin\/(.+)/;
  matcher[0][1];
}
