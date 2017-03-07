#!/bin/groovy

stage('Dev Environment'){ // for display purposes
  node('web-dev') {
    stage('Prepare') { // for display purposes
      // Get some code from a GitHub repository
      git branch: 'dev', credentialsId: '999d62df-f2af-443c-935b-31c16ed196bb', url: 'https://github.com/kisitlikaynaklar/kisitlikaynaklar.github.io'
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
  build job: 'web-stage-test'
}

stage('Production'){
  node('web-prod') {
    stage('Prepare') {
      git branch: 'master', credentialsId: '999d62df-f2af-443c-935b-31c16ed196bb', url: 'https://github.com/kisitlikaynaklar/kisitlikaynaklar.github.io'
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
      sh 'echo cokta guzel oldu pekte guzel oldu taam mi'
}
