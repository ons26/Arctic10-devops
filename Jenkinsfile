pipeline {
    agent any
    environment {
         DOCKER_IMAGE = "benmaaouia/spring-backend"
         DOCKER_TAG = "latest"
        MYSQL_ROOT_PASSWORD = "root"
        MYSQL_DATABASE = "studentdb"
        MYSQL_CONTAINER_NAME = "mysql-student"
    }

    stages {
        stage('Start MySQL') {
            steps {
                script {
                    sh """
                     docker rm -f ${MYSQL_CONTAINER_NAME} || true
                        docker run -d --name ${MYSQL_CONTAINER_NAME} \
                        -e MYSQL_ROOT_PASSWORD=${MYSQL_ROOT_PASSWORD} \
                        -e MYSQL_DATABASE=${MYSQL_DATABASE} \
                        -p 3306:3306 mysql:8.0
                    """
                }
            }
        }

        stage('Wait for MySQL') {
            steps {
                script {
                    sh '''
                        echo "⏳ Waiting for MySQL to be ready..."
                        timeout=60
                        count=0
                        until docker exec mysql-student mysqladmin ping -uroot -proot --silent; do
                            sleep 2
                            count=$((count+2))
                            if [ $count -ge $timeout ]; then
                                echo "❌ MySQL not ready after $timeout seconds"
                                docker logs mysql-student
                                exit 1
                            fi
                        done
                        echo "✅ MySQL is ready!"
                    '''
                }
            }
        }

        stage('Git') {
            steps {
                git branch: 'main', url: 'https://github.com/ons26/Arctic10-devops.git'
            }
        }

        stage('Build') {
            steps {
                sh 'mvn -B -DskipTests clean package'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'mvn test'
            }
        }
          stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
                }
            }
        }
                stage('Push Docker Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh """
                        echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                    """
                }
            }
        }
             stage('Deploy to Kubernetes') {
            steps {
                sh """
                    kubectl apply -f /k8s/mysql-deployment.yaml
                    kubectl apply -f /k8s/backend-deployment.yaml
                """
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('MySonarqube') {
                    withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh 'mvn sonar:sonar -Dsonar.projectKey=arctic10-devops -Dsonar.host.url=http://192.168.177.172:9000 -Dsonar.login=$SONAR_TOKEN'
                    }
                }
            }
        }
    } // fin de stages

    post {
        always {
            // Arrête et supprime le conteneur MySQL pour nettoyage
            sh "docker stop ${MYSQL_CONTAINER_NAME} || true"
            sh "docker rm ${MYSQL_CONTAINER_NAME} || true"
        }
        success {
            echo "Pipeline terminé avec succès ✅"
        }
        failure {
            echo "Le pipeline a échoué ❌"
        }
    }

} // fin de pipeline
