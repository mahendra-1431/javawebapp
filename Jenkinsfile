pipeline {
  agent any
  stages {
    stage('checkout') {
      steps {
        checkout scmGit(branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: 'githubcred', url: 'https://github.com/keyspaceits/javawebapp']])
      }
    }

    stage('Build') {
      steps {
        script {
          try {
            sh "mvn -Dmaven.test.failure.ignore=true clean package"
            slackSend(channel: 'ci-cd-alerts', color: 'good', message: "Build Success! Job Name: ${env.JOB_NAME} Build Num: ${env.BUILD_NUMBER} (${env.BUILD_URL})", teamDomain: 'keyspaceitsgroup', tokenCredentialId: 'slackcred', username: 'suresh')
          } catch (Exception e) {
            slackSend(channel: 'ci-cd-alerts', color: 'danger', message: "Build Failed! Job Name: ${env.JOB_NAME} Build Num: ${env.BUILD_NUMBER} (${env.BUILD_URL})", teamDomain: 'keyspaceitsgroup', tokenCredentialId: 'slackcred', username: 'suresh')
            error 'Build failed'
          }
        }

      }
    }

    stage('CodeQulity') {
      steps {
        withSonarQubeEnv('SonarQubeServer') {
          sh 'mvn clean install -f pom.xml sonar:sonar'
        }

      }
    }

    stage('Prod Aprove') {
      steps {
        echo 'taking approval from Prod manager for Prod Deployment'
        timeout(time: 7, unit: 'DAYS') {
          input(message: 'Do you approve Prod Deployment?', submitter: 'admin')
        }

      }
    }

    stage('deploy to prod') {
      steps {
        script {
          try {
            deploy adapters: [tomcat9(alternativeDeploymentContext: '', credentialsId: 'tomcatdeployer', path: '', url: 'http://172.25.0.229:8080')], contextPath: null, war: '**/*.war'
            slackSend(channel: 'ci-cd-alerts', color: 'good', message: "Prod Deployment was successful! here is the Job Name: ${env.JOB_NAME} Build Num: ${env.BUILD_NUMBER} (${env.BUILD_URL})", teamDomain: 'keyspaceitsgroup', tokenCredentialId: 'slackcred')
          } catch (Exception e) {
            slackSend(channel: 'ci-cd-alerts', color: 'danger', message: "Prod Deployment Failed! here is the  Job Name: ${env.JOB_NAME} Build Num: ${env.BUILD_NUMBER} (${env.BUILD_URL})", teamDomain: 'keyspaceitsgroup', tokenCredentialId: 'slackcred')
            error 'Deployment failed'
          }
        }

      }
    }

  }
  tools {
    maven 'Maven3.910'
  }
}
