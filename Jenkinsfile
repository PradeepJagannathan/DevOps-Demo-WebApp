pipeline {
  agent any
  environment {
    buildnum = currentBuild.getNumber()
    gitURL= "https://github.com/PradeepJagannathan/DevOps-Demo-WebApp.git"
    sonarPath = 'http://52.148.170.197:9000'
    sonarInclusion = '**/test/java/servlet/createpage_junit.java'
    sonarExclusion = '**/test/java/servlet/createpage_junit.java'
    tomcatTestURL= "http://40.71.32.30:8080"
    tomcatProdURL= "http://52.255.182.18:8080"
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
        slackSend channel: '#alerts', message: 'Checking code from Git'
      }
    }
    stage('compile'){
      steps {
 //       def mvnHome = tool name: 'maven', type: maven
        sh "mvn compile"
        slackSend channel: '#alerts', message: 'Compiling'
      }
    }
    stage ('static code analysis') {
      steps {
        withSonarQubeEnv(credentialsId: 'sonar',installationName:'sonarserver') {
          sh 'mvn clean install sonar:sonar -Dsonar.host.url = ${sonarPath} -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=${sonarInclusion} -Dsonar.test.exclusion=${sonarExclusion} -Dsonar.login=admin -Dsonar.password=admin' 
        }
      }
    }
    stage ('Deploy to Test') {
      steps {
        sh 'mvn clean package'
        deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: "${tomcatTestURL}")], contextPath: '/QAWebapp', war: '**/*.war'
        slackSend channel: '#alerts', message: 'Code is deployed to test'
      }
    }
    stage ('Artifact') {
      steps {
        rtUpload(serverId: 'artifactory')
        rtPublishBuildInfo (serverId: 'artifactory')
        slackSend channel: '#alerts', message: 'Deployed code to arifact'
      }
    }        
    
    stage ('Perform UI Test') {
      steps {
        sh 'mvn test'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test', reportTitles: ''])
        slackSend channel: '#alerts', message: 'Generating UI report'
      }
    }
//    stage("Performance Test") {
//      steps {
//        blazeMeterTest credentialsId: 'blazemeter', testId: '8578936.taurus', workspaceId: '667789'
//        slackSend channel: '#alerts', message: 'Performanance test'
//      }
//    }
    stage ('Deploy to Prod') {
      steps {
        sh 'mvn clean install'
        deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: "${tomcatProdURL}")], contextPath: '/ProdWebapp', war: '**/*.war'
        slackSend channel: '#alerts', message: 'Code is deployed to prod'
      }
    }
    
    stage ('Sanity Test') {
      steps {
        sh 'mvn test'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test', reportTitles: ''])
        slackSend channel: '#alerts', message: 'Generating Sanity report'
      }
    }

  }
}
    
