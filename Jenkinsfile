pipeline {

    agent any


    tools { 
        maven 'my-maven' 
        dockerTool  'docker'
    }
    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root')
    }
    stages {

        stage('Build with Maven') {
            steps {
                sh 'mvn --version'
                sh 'java -version'
                sh 'mvn clean package -Dmaven.test.failure.ignore=true'
            }
        }

        stage('Packaging/Pushing image') {
    steps {
        script {
            try {
                withDockerRegistry(credentialsId: 'dockerhubAccount', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t tuannguyen333/springboot .'
                    sh 'docker push tuannguyen333/springboot'
                }
            } catch (Exception e) {
                echo "Error occurred: ${e.getMessage()}"
                error "Failed to build and push Docker image"
            }
        }
    }
}

    stage('Test Docker') {
    steps {
        sh 'docker --version'
    }
}

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop tuannguyen-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm tuannguyen-mysql-data || echo "no volume"'

                sh "docker run --name tuannguyen-mysql --rm --network dev -v tuannguyen-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
                sh 'sleep 20'
                sh "docker exec -i tuannguyen-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull tuannguyen333/springboot'
                sh 'docker container stop tuannguyen333-springboot || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name tuannguyen333-springboot -p 8081:8080 --network dev tuannguyen333/springboot'
            }
        }
 
    }
    post {
        // Clean after build
        always {
            cleanWs()
        }
    }
}
