node {
  stage('Preparation') {
      dir('git-repo') {
        checkout([$class: 'GitSCM', branches: [[name: '*/*']], userRemoteConfigs: [[url: 'https://github.com/pdincau/testrepo.git']]])

        env.BRANCH_NAME = sh(script: 'git rev-parse --abbrev-ref HEAD', returnStdout: true)
        echo env.BRANCH_NAME

        sh 'git rev-parse --abbrev-ref HEAD > GIT_BRANCH'
        git_branch = readFile('GIT_BRANCH').trim()
        echo git_branch
     }
   }
   stage('sync'){
   sh '''
      DESTINATION_STREAM=$BRANCH_NAME
      RTC_URL="https://10.0.0.74:9443/ccm"
      RTC_USERNAME="valentina"
      RTC_PASSWORD="valentina"
      REMOTE_WORKSPACE="sync-workspace$BUILD_NUMBER"
      LOCAL_WORKSPACE="local-sync-workspace"

      PROJECT_NAME="testrepo"

      /opt/scmtools/eclipse/scm login -u $RTC_USERNAME -P $RTC_PASSWORD -r $RTC_URL
      /opt/scmtools/eclipse/scm create workspace -r $RTC_URL -s $DESTINATION_STREAM $REMOTE_WORKSPACE
      /opt/scmtools/eclipse/scm load -r $RTC_URL -f -d $LOCAL_WORKSPACE --all $REMOTE_WORKSPACE

      rsync -av --progress git-repo/* $LOCAL_WORKSPACE/SRC/$PROJECT_NAME --exclude .git --delete

      /opt/scmtools/eclipse/scm checkin --comment "synch commit" $LOCAL_WORKSPACE
      /opt/scmtools/eclipse/scm deliver -d $LOCAL_WORKSPACE --source $REMOTE_WORKSPACE -r $RTC_URL
      /opt/scmtools/eclipse/scm delete workspace -r $RTC_URL $REMOTE_WORKSPACE'''
   }
}
