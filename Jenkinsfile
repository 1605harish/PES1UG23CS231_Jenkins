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

                # give containers a moment to start
                sleep 2
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                set -e
                docker rm -f nginx-lb || true

                # If port 80 is already in use, change this to -p 8081:80 and browse http://localhost:8081
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 80:80 \
                  nginx

                # give nginx a moment to start before copying config
                sleep 2

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
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
