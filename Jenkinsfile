pipeline {
    agent any

    stages {
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
          stage('Test') {
            steps {
                // Cette commande va exécuter les tests Maven
                sh 'mvn test'
            }
        }
    }
}
