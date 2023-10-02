pipeline {

    agent any

    environment {
        MYSQL_ROOT_LOGIN = credentials('mysql-root-login')
    }
    stages {

        stage('Packaging/Pushing imagae') {

            steps {
               withDockerRegistry(credentialsId: 'dockerhub', url: 'https://www.docker.com/v1') {
                   sh 'docker build -t tannv95/springbooDemo .'
                   sh 'docker push  tannv95/springbooDemo'
               }
            }
        }

        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop khalid-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm khalid-mysql-data || echo "no volume"'

                sh "docker run --name khalid-mysql --rm --network dev -v khalid-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_LOGIN_PSW} -e MYSQL_DATABASE=db_example  -d mysql:8.0 "
                sh 'sleep 20'
                sh "docker exec -i khalid-mysql mysql --user=root --password=${MYSQL_ROOT_LOGIN_PSW} < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull tannv95/springbooDemo'
                sh 'docker container stop tannv95-springbooDemo || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name tannv95-springbooDemo -p 8081:8080 --network dev tannv95/springbooDemo'
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
