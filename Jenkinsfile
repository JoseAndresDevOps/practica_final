pipeline{
  agent {
        kubernetes {
        yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: joseandresdevops/nuevojava:4.0
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: docker-socket-volume
    securityContext:
      privileged: true
  volumes:
  - name: docker-socket-volume
    hostPath:
      path: /var/run/docker.sock
      type: Socket
    command:
    - sleep
    args:
    - infinity
        '''
            defaultContainer 'shell'
        }
    }

  environment {
    registryCredential='docker-hub-credentials'
    registryBackend = 'joseandresdevops/spring-boot-app'
  }

    stages {

        stage("test"){
            steps{
                sh "mvn test"
                jacoco()
                junit "target/surefire-reports/*.xml"
            }
        }

        stage('build') {
            steps {
                sh 'mvn --version'
                sh "mvn clean package -DskipTest"
                //archiveArtifcats artifacts: '**/target/*.jar', fingerprint: true
            }
        }
        
        stage('Push Image to Docker Hub') {
            steps {
                script {
                    dockerImage = docker.build registryBackend + ":$BUILD_NUMBER"
                    docker.withRegistry( '', registryCredential) {
                    dockerImage.push()
                    }
                }
            }
        }

        //stage('Push Image latest to Docker Hub') {
        //    steps {
        //        script {
        //            dockerImage = docker.build registryBackend + ":latest"
        //            docker.withRegistry( '', registryCredential) {
        //            dockerImage.push()
        //            }
        //        }
        //    }
        //}

        //stage('NPM build') {
        //    steps {
        //        script {
        //            sh 'npm install'
        //            sh 'npm run build'
        //        }
        //    }
        //}
        //stage('SonarQube analysis') {
        //    steps {
        //        withSonarQubeEnv(credentialsId: "sonarqube-server", installationName: "sonarqube-server"){
        //        sh 'npm run sonar'
        //        }
        //    }
        //}

    }
}
