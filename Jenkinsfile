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
    }
     post {
        success {
            // Envoie un email quand le build réussit
            mail to: 'ons.benmaaouia@esprit.tn',
                 subject: "Build réussi: ${currentBuild.fullDisplayName}",
                 body: "Le build ${currentBuild.fullDisplayName} a réussi. Consulte Jenkins pour plus de détails."
        }
}
