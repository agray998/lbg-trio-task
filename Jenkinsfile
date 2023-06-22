pipeline {
  agent any
  
  environment {
    DB_PASSWORD = credentials('DB_PASSWORD')
  }
  stages {
    stage('sonarqube scan') {
      steps {
        withSonarQubeEnv('my-sonar') {
           sh 'sonar-scanner'
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
