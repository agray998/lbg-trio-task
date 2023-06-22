pipeline {
  agent any
  
  environment {
    DB_PASSWORD = credentials('DB_PASSWORD')
  }
  
  stages {
    stage('sonarqube scan') {
      def sonarHome = tool 'SonarScanner 4.8';
      steps {
        withSonarQubeEnv('my-sonar') {
           sh '${sonarHome}/bin/sonar-scanner'
        }
      }
    }
    stage('build') {
      steps {
      	sh 'docker-compose build'
      }
    }
    stage('run') {
      steps {
      	sh 'docker-compose up -d'
    	}
     }
  }
}
