pipeline {
  agent any
  environment {
    buildnum = currentBuild.getNumber()
    gitURL= "https://github.com/PradeepJagannathan/DevOps-Demo-WebApp.git"
    tomcatTestURL= "http://40.71.32.30:8080"
    tomcatProdURL= "http://52.225.182.18:8080"
    uiPath = "\\functionaltest\\target\\surefire-reports"
    sanityPath="\\Acceptance\\target\\surefire-reports"
  }
  
  tools {
    maven 'maven'
    jdk 'jdk'
  }
  
  stages {
    stage('checkout'){
      steps {
        checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: "${gitURL}"]]])
        slackSend channel: '#alerts', message: 'Testing Slack'
      }
    }
    stage('compile'){
      steps {
 //       def mvnHome = tool name: 'maven', type: maven
        sh "mvn compile"
      }
    }
//    stage ('static code analysis') {
//      steps {
//        withSonarQubeEnv ('sonarqube') {
//          sh 'mvn clean package sonar:sonar \
//          -Dsonar.sources=. 
//        }
//      }
//    }
    stage ('Deploy to Test') {
      steps {
        sh 'mvn clean package'
        deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: "${tomcatTestURL}")], contextPath: '/QAWebapp', war: '**/*.war'
      }
    }
    stage ('Artifact') {
      steps {
        rtUpload(serverId: 'artifactory')
        rtPublishBuildInfo (serverId: 'artifactory')
      }
    }        
    
    stage ('Perform UI Test') {
      steps {
        sh 'mvn test'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test', reportTitles: ''])
      }
    }
//    stage("Performance Test") {
//      steps {
//        blazeMeterTest credentialsId: 'blazemeter', testId: '8578936.taurus', workspaceId: '667789'
//      }
//    }
    stage ('Deploy to Prod') {
      steps {
        sh 'mvn clean install'
        deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: "${tomcatProdURL}")], contextPath: '/ProdWebapp', war: '**/*.war'
      }
    }
    
    stage ('Sanity Test') {
      steps {
        sh 'mvn test'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test', reportTitles: ''])
      }
    }

  }
}
    
