pipeline {
    agent any

    stages {
        stage('Build Backend Image') {
            steps {
                sh '''
                set -e
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                set -e
                docker network create app-network || true
                docker rm -f backend1 backend2 || true

                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app

                # wait for services + docker DNS to be ready
                sleep 5
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                set -e
                docker rm -f nginx-lb || true

                # If port 80 is busy, change to: -p 8081:80 and browse http://localhost:8081
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                # wait for nginx container start
                sleep 3

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # wait before config test to avoid DNS race
                sleep 2

                docker exec nginx-lb nginx -t
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
