#!groovy

String gitUrl = 'https://github.com/kisitlikaynaklar/kisitlikaynaklar.github.io'
String gitID = '999d62df-f2af-443c-935b-31c16ed196bb'

def WebTest = {
  sh 'echo $HOSTNAME'
  sh '/usr/local/bin/web-test.sh'
}

def WebInstall = {
  sh "bundle install"
}

def WebBuild = {
  sh "jekyll build"
  sh '''
  rm -rf /var/www/html/*
  mv _site/* /var/www/html/
  cp htaccess /var/www/html/.htaccess
  chown -R apache:apache /var/www/html
  '''
}

def GitMerge = {
  checkout([
    $class: 'GitSCM',
    branches: [[name: 'refs/heads/dev']],
    userRemoteConfigs: [[
      credentialsId: gitID,
      name: 'origin',
      url: gitUrl
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
      localBranch: 'dev'
    ]]
  ])
  withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: gitID, usernameVariable: 'gitUsername', passwordVariable: 'gitPassword']]) {
    sh('git push https://${gitUsername}:${gitPassword}@github.com/kisitlikaynaklar/kisitlikaynaklar.github.io')
}
}

stage('Merge'){
  node{
    GitMerge()
  }
}

stage('Dev Environment'){ // for display purposes
  node('web-dev') {
    stage('Prepare') { // for display purposes
      git branch: 'dev', credentialsId: gitID, url: gitUrl
    }
    stage('Install') {
      WebInstall()
    }
    stage('Build') {
      WebBuild()
    }
    step([$class: 'WsCleanup'])
  }
}

stage('Dev-Test'){
  node('web-dev') {
    WebTest()
  }
}

stage('Stage Environment'){
  node('web-dev') {
    stage('Prepare') { // for display purposes
      git branch: 'dev', credentialsId: gitID, url: gitUrl
    }
    stage('Install') {
      WebInstall()
    }
    stage('Build') {
      WebBuild()
    }
    step([$class: 'WsCleanup'])
  }
}

stage('Stage Test'){
  node('web-stage') {
    WebTest()
  }
}

stage('Pre-prod Tests') {
  parallel 'web-dev-test':{
    node('web-dev'){
      WebTest()
    }
  }, 'web-stage-test':{
    node('web-stage'){
      WebTest()
    }
  }
}

timeout(time:1, unit:'MINUTES') {
    input message:'Approve deployment?', ok: 'Go ahead'
}

stage('Merge'){
  node{
    GitMerge()
    step([$class: 'WsCleanup'])
  }
}

stage('Production'){
  node('web-prod') {
    stage('Prepare') {
      git branch: 'master', credentialsId: gitID, url: gitUrl
    }
    stage('Install') {
      WebInstall()
    }
    stage('Build') {
      WebBuild()
    }
    step([$class: 'WsCleanup'])
  }
}

stage('Tests') {
  parallel 'web-dev-test':{
    node('web-dev'){
      WebTest()
    }
  }, 'web-stage-test':{
    node('web-stage'){
      WebTest()
    }
  }, 'web-prod-test':{
    node('web-prod'){
      WebTest()
    }
  }
}
stage('Result') {
  node(){
    sh 'echo cokta guzel oldu pekte guzel oldu taam mi'
  }
}
