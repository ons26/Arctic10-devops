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
         stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('MySonarqube') {
            withCredentials([string(credentialsId: 'sonar-token', variable: 'SONAR_TOKEN')]) {
                sh 'mvn sonar:sonar -Dsonar.projectKey=arctic10-devops -Dsonar.host.url=http://192.168.177.172:9000 -Dsonar.login=$SONAR_TOKEN'
            }
        }
    }
}

    }

}
