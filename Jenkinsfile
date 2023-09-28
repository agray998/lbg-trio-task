pipeline {
    agent any
    stages {
        stage('Build images') {
            steps {
                sh '''
                docker build -t gcr.io/lbg-mea-14/ag-trio-task-db:latest ./db
                docker build -t gcr.io/lbg-mea-14/ag-trio-task-db:${BUILD_NUMBER} ./db
                docker build -t gcr.io/lbg-mea-14/ag-trio-task-app:latest ./flask-app
                docker build -t gcr.io/lbg-mea-14/ag-trio-task-app:${BUILD_NUMBER} ./flask-app
                docker build -t gcr.io/lbg-mea-14/ag-trio-task-rp:latest ./nginx
                docker build -t gcr.io/lbg-mea-14/ag-trio-task-rp:${BUILD_NUMBER} ./nginx
                '''
            }
        }
        stage('Push Images') {
            steps {
                sh '''
                docker push gcr.io/lbg-mea-14/ag-trio-task-db
                docker push gcr.io/lbg-mea-14/ag-trio-task-db:${BUILD_NUMBER}
                docker push gcr.io/lbg-mea-14/ag-trio-task-app
                docker push gcr.io/lbg-mea-14/ag-trio-task-app:${BUILD_NUMBER}
                docker push gcr.io/lbg-mea-14/ag-trio-task-rp
                docker push gcr.io/lbg-mea-14/ag-trio-task-rp:${BUILD_NUMBER}
                '''
            }
        }
        stage('Cleanup Jenkins') {
            steps {
                sh '''
                docker rmi gcr.io/lbg-mea-14/ag-trio-task-db
                docker rmi gcr.io/lbg-mea-14/ag-trio-task-db:${BUILD_NUMBER}
                docker rmi gcr.io/lbg-mea-14/ag-trio-task-app
                docker rmi gcr.io/lbg-mea-14/ag-trio-task-app:${BUILD_NUMBER}
                docker rmi gcr.io/lbg-mea-14/ag-trio-task-rp
                docker rmi gcr.io/lbg-mea-14/ag-trio-task-rp:${BUILD_NUMBER}
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
                docker pull gcr.io/lbg-mea-14/ag-trio-task-app
                docker pull gcr.io/lbg-mea-14/ag-trio-task-db
                docker pull gcr.io/lbg-mea-14/ag-trio-task-rp
                docker network inspect trio && sleep 1 || docker network create trio
                docker volume inspect trio && sleep 1 || docker volume create trio
                
                docker stop mysql && (docker rm mysql) || (docker rm mysql && sleep 1 || sleep 1)
                docker stop flask-app && (docker rm flask-app) || (docker rm flask-app && sleep 1 || sleep 1)
                docker stop nginx && (docker rm nginx) || (docker rm nginx && sleep 1 || sleep 1)

                docker run -d -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} -v trio:/var/lib/mysql --network trio --name mysql gcr.io/lbg-mea-14/ag-trio-task-db
                docker run -d -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} --network trio --name flask-app gcr.io/lbg-mea-14/ag-trio-task-app
                docker run -d -p 80:80 --network trio --name nginx gcr.io/lbg-mea-14/ag-trio-task-rp
                '''
            }
        }
    }
}
