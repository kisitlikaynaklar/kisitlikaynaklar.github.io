#!groovy

String giturl = 'https://github.com/kisitlikaynaklar/kisitlikaynaklar.github.io'
String gitID = '999d62df-f2af-443c-935b-31c16ed196bb'
def credentialsId = scm.userRemoteConfigs[0].credentialsId

stage('Dev Environment'){ // for display purposes
  node('web-dev') {
    stage('Prepare') { // for display purposes
      // Get some code from a GitHub repository
      git branch: 'dev', credentialsId: gitID, url: giturl
    }
    stage('Install') {
      sh "bundle install"
    }
    stage('Build') {
      sh "jekyll build"
      sh '''
      rm -rf /var/www/html/*
      mv _site/* /var/www/html/
      cp htaccess /var/www/html/.htaccess
      chown -R apache:apache /var/www/html
      '''
    }
    step([$class: 'WsCleanup'])
  }
}

stage('Dev-Test'){
  node('web-dev') {
    sh 'echo $HOSTNAME && /usr/local/bin/web-test.sh'
  }
}

stage('Stage Environment'){
  build job: 'web-stage-serve'
}

stage('Stage Test'){
  node('web-stage') {
    sh 'echo $HOSTNAME && /usr/local/bin/web-test.sh'
  }
}

stage('Pre-prod Tests') {
  parallel 'web-dev-test':{
    node('web-dev'){ sh 'echo $HOSTNAME && /usr/local/bin/web-test.sh' }
  }, 'web-stage-test':{
    node('web-stage'){ sh 'echo $HOSTNAME &&  /usr/local/bin/web-test.sh' }
  }
}

timeout(time:1, unit:'MINUTES') {
    input message:'Approve deployment?', ok: 'Go ahead'
}

stage('Merge'){
  node{
    checkout([
      $class: 'GitSCM',
      branches: [[name: 'refs/heads/dev']],
      userRemoteConfigs: [[
        credentialsId: gitID,
        name: 'origin',
        url: giturl
      ]],
      extensions: [
      [
        $class: 'PreBuildMerge',
        options: [
          fastForwardMode: 'FF',
          mergeRemote: 'origin',
          mergeStrategy: 'MergeCommand.Strategy',
          mergeTarget: 'master'
        ]
      ],
      [
        $class: 'LocalBranch',
        localBranch: 'master'
      ]]
    ])
  }
}


stage('Production'){
  node('web-prod') {
    stage('Prepare') {
      git branch: 'master', credentialsId: gitID, url: giturl
    }
    stage('Install') {
      sh "bundle install"
    }
    stage('Build') {
      sh "jekyll build"
      sh '''
      rm -rf /var/www/html/*
      mv _site/* /var/www/html/
      cp htaccess /var/www/html/.htaccess
      chown -R apache:apache /var/www/html
      '''
    }
    step([$class: 'WsCleanup'])
  }
}

stage('Tests') {
  parallel 'web-dev-test':{
    node('web-dev'){ sh 'echo $HOSTNAME && /usr/local/bin/web-test.sh' }
  }, 'web-stage-test':{
    node('web-stage'){ sh 'echo $HOSTNAME &&  /usr/local/bin/web-test.sh' }
  }, 'web-prod-test':{
    node('web-prod'){ sh 'echo $HOSTNAME &&  /usr/local/bin/web-test.sh' }
  }
}
stage('Result') {
  node(){
    sh 'echo cokta guzel oldu pekte guzel oldu taam mi'
  }
}
