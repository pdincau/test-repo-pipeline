node {
  env.PROJECT_NAME="testrepo"

  env.RTC_URL="https://10.0.0.74:9443/ccm"
  env.RTC_USERNAME="valentina"
  env.RTC_PASSWORD="valentina"
  env.REMOTE_WORKSPACE="sync-workspace$BUILD_NUMBER"
  env.LOCAL_WORKSPACE="local-sync-workspace"

  stage('Preparation') {
      dir('git-repo') {
        checkout([$class: 'GitSCM', branches: [[name: '*/*']], userRemoteConfigs: [[url: "https://github.com/pdincau/${env.PROJECT_NAME}.git"]]])

        def completeName = sh(script: 'git name-rev --name-only HEAD', returnStdout: true)
        def matcher = completeName =~ /remotes\/origin\/(.+)/;
        env.BRANCH_NAME = matcher[0][1];
     }
   }
   stage('sync'){
      sh '''
        /opt/scmtools/eclipse/scm login -u $RTC_USERNAME -P $RTC_PASSWORD -r $RTC_URL
        /opt/scmtools/eclipse/scm create workspace -r $RTC_URL -s $BRANCH_NAME $REMOTE_WORKSPACE
      '''

      sh '''
        /opt/scmtools/eclipse/scm load -r $RTC_URL -f -d $LOCAL_WORKSPACE --all $REMOTE_WORKSPACE

        rsync -av --progress git-repo/* $LOCAL_WORKSPACE/SRC/$PROJECT_NAME --exclude .git --delete

        /opt/scmtools/eclipse/scm checkin --comment "synch commit" $LOCAL_WORKSPACE
        /opt/scmtools/eclipse/scm deliver -d $LOCAL_WORKSPACE --source $REMOTE_WORKSPACE -r $RTC_URL
      '''

      sh '''
        /opt/scmtools/eclipse/scm delete workspace -r $RTC_URL $REMOTE_WORKSPACE
      '''
   }
}
