pipeline {
    agent any
    
    stages {
        stage('Build Backend Image') {
            steps {
                sh '''
                    echo "Building backend image..."
                    docker rmi -f backend-app || true
                    docker build -t backend-app ./backend
                    docker images | grep backend-app
                '''
            }
        }
        
        stage('Deploy Backend Containers') {
            steps {
                sh '''
                    echo "Creating network if not exists..."
                    docker network create app-network || true
                    
                    echo "Cleaning old backend containers..."
                    docker rm -f backend1 backend2 || true
                    
                    echo "Starting backend1..."
                    docker run -d --name backend1 --network app-network backend-app
                    
                    echo "Starting backend2..."
                    docker run -d --name backend2 --network app-network backend-app
                    
                    echo "Waiting for backends to start..."
                    sleep 5
                    
                    echo "Checking running backends..."
                    docker ps --filter "name=backend" --format '{{.ID}} {{.Names}} {{.Status}}'
                '''
            }
        }
        
        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                    echo "Cleaning old NGINX container..."
                    docker rm -f nginx-lb || true
                    
                    echo "Starting NGINX load balancer..."
                    docker run -d \
                        --name nginx-lb \
                        --network app-network \
                        -p 8081:80 \
                        nginx:latest
                    
                    echo "Waiting for NGINX to start..."
                    sleep 5
                    
                    echo "Copying custom configuration..."
                    docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
                    
                    echo "Reloading NGINX..."
                    docker exec nginx-lb nginx -s reload
                    
                    echo "Checking NGINX status..."
                    docker logs nginx-lb --tail 15
                '''
            }
        }
    }
    
    post {
        always {
            echo 'Pipeline finished.'
            sh 'docker ps --format "table {{.ID}}\\t{{.Names}}\\t{{.Status}}\\t{{.Ports}}"'
        }
        success {
            echo 'Pipeline executed successfully. Access load balancer at http://localhost:8081'
        }
        failure {
            echo 'Pipeline failed. Check console logs above.'
        }
    }
}
