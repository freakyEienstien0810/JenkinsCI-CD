pipeline {
    agent {
       label "master" 
    }
    tools {
      jdk "JAVA_HOME"
      maven "Maven-3.6.1"
    }
    stages {
        stage('Build') {
            steps {
                bat "mvn clean package"
            }
        }
    }  
}
