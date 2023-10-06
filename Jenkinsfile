pipeline {

    agent any

    tools {
            maven 'my-maven'
        }

     environment {
                MYSQL_ROOT_USERNAME = ''
                MYSQL_ROOT_PASSWORD = ''
        }


    stages {

    stages {
         stage('Get MySQL Credentials') {
                     steps {
                         script {
                             withCredentials([usernamePassword(credentialsId: 'mysql-root-login', usernameVariable: 'MYSQL_ROOT_USERNAME', passwordVariable: 'MYSQL_ROOT_PASSWORD')]) {
                                 // Do something with MYSQL_ROOT_USERNAME and MYSQL_ROOT_PASSWORD
                             }
                         }
                     }
                 }


        stage('Packaging/Pushing images') {
            steps {
                withDockerRegistry(credentialsId: 'dockerhub', url: 'https://index.docker.io/v1/') {
                    sh 'docker build -t tannv95/dockerdemo .'
                    sh 'docker push tannv95/dockerdemo'
                    }
                }
        }


        stage('Deploy MySQL to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull mysql:8.0'
                sh 'docker network create dev || echo "this network exists"'
                sh 'docker container stop tannv-mysql || echo "this container does not exist" '
                sh 'echo y | docker container prune '
                sh 'docker volume rm tannv-mysql-data || echo "no volume"'

                sh "docker run --name tannv-mysql --rm --network dev -v tannv-mysql-data:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=1 -e MYSQL_DATABASE=db_example  -d mysql:8.0"
                sh 'sleep 20'
                sh "docker exec -i tannv-mysql mysql --user=root --password=1 < script"
            }
        }

        stage('Deploy Spring Boot to DEV') {
            steps {
                echo 'Deploying and cleaning'
                sh 'docker image pull tannv95/dockerdemo'
                sh 'docker container stop tannv95-dockerdemo || echo "this container does not exist" '
                sh 'docker network create dev || echo "this network exists"'
                sh 'echo y | docker container prune '

                sh 'docker container run -d --rm --name tannv95-dockerdemo -p 8081:8080 --network dev tannv95/dockerdemo'
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
