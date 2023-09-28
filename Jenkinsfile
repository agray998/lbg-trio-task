pipeline {
    agent any
    stages {
        stage('Build images') {
            steps {
                sh '''
                docker build -t agray998/trio-task-db:latest ./db
                docker build -t agray998/trio-task-db:${BUILD_NUMBER} ./db
                docker build -t agray998/trio-task-app:latest ./flask-app
                docker build -t agray998/trio-task-app:${BUILD_NUMBER} ./flask-app
                docker build -t agray998/trio-task-rp:latest ./nginx
                docker build -t agray998/trio-task-rp:${BUILD_NUMBER} ./nginx
                '''
            }
        }
        stage('Push Images') {
            steps {
                sh '''
                docker push agray998/trio-task-db
                docker push agray998/trio-task-db:${BUILD_NUMBER}
                docker push agray998/trio-task-app
                docker push agray998/trio-task-app:${BUILD_NUMBER}
                docker push agray998/trio-task-rp
                docker push agray998/trio-task-rp:${BUILD_NUMBER}
                '''
            }
        }
        stage('Cleanup Jenkins') {
            steps {
                sh '''
                docker rmi agray998/trio-task-db
                docker rmi agray998/trio-task-db:${BUILD_NUMBER}
                docker rmi agray998/trio-task-app
                docker rmi agray998/trio-task-app:${BUILD_NUMBER}
                docker rmi agray998/trio-task-rp
                docker rmi agray998/trio-task-rp:${BUILD_NUMBER}
                '''
            }
        }
        stage('Deploy Containers') {
            environment {
                MYSQL_ROOT_PASSWORD = credentials('MYSQL_ROOT_PASSWORD')
            }
            steps {
                sh '''
                ssh jenkins@piers-appserver << EOF
                docker pull agray998/trio-task-app
                docker pull agray998/trio-task-db
                docker pull agray998/trio-task-rp
                docker network inspect trio && sleep 1 || docker network create trio
                docker volume inspect trio && sleep 1 || docker volume create trio
                
                docker stop mysql && (docker rm mysql) || (docker rm mysql && sleep 1 || sleep 1)
                docker stop flask-app && (docker rm flask-app) || (docker rm flask-app && sleep 1 || sleep 1)
                docker stop nginx && (docker rm nginx) || (docker rm nginx && sleep 1 || sleep 1)

                docker run -d -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} -v trio:/var/lib/mysql --network trio --name mysql agray998/trio-task-db
                docker run -d -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} --network trio --name flask-app agray998/trio-task-app
                docker run -d -p 80:80 --network trio --name nginx agray998/trio-task-rp
                '''
            }
        }
    }
}
