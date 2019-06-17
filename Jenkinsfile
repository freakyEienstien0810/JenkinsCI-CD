pipeline {
    agent {
       label "master" 
    }
    tools {
      jdk "JAVA_HOME"
      maven "Maven-3.6.1"
    }
    environment {
    //Use Pipeline Utility Steps plugin to read information from pom.xml into env variables
    IMAGE = readMavenPom().getArtifactId()
    VERSION = readMavenPom().getVersion()
    }
    stages {
        stage('Build') {
            steps {
                echo 'IMAGE - $IMAGE and Version - $VERSION'
                //bat "mvn clean package"
            }
        }
    }  
}
