pipeline {
  agent any
  environment {
    buildnum = currentBuild.getNumber()
    gitURL= "https://github.com/PradeepJagannathan/DevOps-Demo-WebApp.git"
    sonarPath = "http://52.156.113.141:9000"
    sonarInclusion = '**/test/java/servlet/createpage_junit.java'
    sonarExclusion = '**/test/java/servlet/createpage_junit.java'
    tomcatTestURL= "http://52.152.232.36:8080"
    tomcatProdURL= "http://13.82.143.159:8080"
    uiPath = "\\functionaltest\\target\\surefire-reports"
    sanityPath="\\Acceptance\\target\\surefire-reports"
  }
  
  tools {
    maven 'maven'
    jdk 'jdk'
  }
  
  stages {
    stage ('Initiation') {
      steps {
        slackSend channel: '#alerts', message: 'Jenkins Build ' +"${buildnum}" +'Initiated'
      }
    }
      
    stage('checkout'){
      steps {
        try {
          checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: "${gitURL}"]]])
          slackSend channel: '#alerts', message: 'Code checked out from Git'
        }
        catch (exc) {
          slackSend channel: '#alerts', message: 'Code check out from Git failed'
        }
      }
    }
     
    stage('compile'){
      steps {
        def mvnHome = tool name: 'maven', type: maven
        sh "mvn compile"
        slackSend channel: '#alerts', message: 'Application Compiled'
      }
    }
    stage ('static code analysis') {
      steps {
        withSonarQubeEnv(credentialsId: 'sonar',installationName:'sonarserver') {
          sh "mvn clean compile sonar:sonar -Dsonar.host.url=${sonarPath} -Dsonar.sources=. -Dsonar.tests=. -Dsonar.inclusions=${sonarInclusion} -Dsonar.test.exclusions=${sonarExclusion} -Dsonar.login=admin -Dsonar.password=admin"
          slackSend channel: '#alerts', message: 'Static code analysis is complete'
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
        slackSend channel: '#alerts', message: 'Deployed code to artifact'
      }
    }        
    
    stage ('Perform UI Test') {
      steps {
        sh 'mvn test'
        publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test', reportTitles: ''])
        slackSend channel: '#alerts', message: 'Generated UI test report'
      }
    }
    stage("Performance Test") {
      steps {
        blazeMeterTest credentialsId: 'Blazemeter', testId: '8510506.taurus', workspaceId: '650230'
        slackSend channel: '#alerts', message: 'Performanance test is complete'
      }
    }
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
        slackSend channel: '#alerts', message: 'Sanity test is complete'
      }
    }
    
    stage ('Completion') {
      steps {
        slackSend channel: '#alerts', message: 'Jenkins Build ' +"${buildnum}" +' SUCCESS!!'
      }
    }
  }
  post {
        success {
          slackSend channel: '#alerts', message: 'Build success'
        }
        failure {
          slackSend channel: '#alerts', message: 'Build Failed'
        }
    } 
}
    
