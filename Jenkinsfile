pipeline{
  agent {
        kubernetes {
        yaml '''
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: shell
    image: joseandresdevops/nuevojava:6.0
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

        stage('build') {
            steps {
                //*sh 'mvn --version'
                sh "mvn clean package -DskipTest"
                //*sh "mvn compile"
                //*sh "mvn package"
                //archiveArtifcats artifacts: '**/target/*.jar', fingerprint: true
            }
        }
        
        stage("test"){
            steps{
                sh "mvn test"
                //*junit "target/surefire-reports/*.xml"
                //*jacoco()
            }
        }

/********************************************** FUNCIONA SUBIR IMAGEN DOCKER
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

        stage('Push Image latest to Docker Hub') {
            steps {
                script {
                    dockerImage = docker.build registryBackend + ":latest"
                    docker.withRegistry( '', registryCredential) {
                    dockerImage.push()
                    }
                }
            }
        }
********************************************/

        stage ("Setup Jmeter") {
            steps{
                script {

                    if(fileExists("jmeter-docker")){
                        sh 'rm -r jmeter-docker'
                    }

                    sh 'git clone https://github.com/joseandresdevops/jmeter-docker.git'

                    dir('jmeter-docker') {

                        if(fileExists("apache-jmeter-5.5.tgz")){
                        sh 'rm -r apache-jmeter-5.5.tgz'
                        }
                    sh 'wget https://dlcdn.apache.org//jmeter/binaries/apache-jmeter-5.5.tgz'
                    sh 'tar xvf apache-jmeter-5.5.tgz'
                    sh 'cp plugins/*.jar apache-jmeter-5.5/lib/ext'
                    sh 'mkdir test'
                    sh 'mkdir apache-jmeter-5.5/test'
                    sh 'cp ../src/main/resources/*.jmx apache-jmeter-5.5/test/'
                    sh 'chmod +775 ./build.sh && chmod +775 ./run.sh && chmod +775 ./entrypoint.sh'
                    sh 'rm -r apache-jmeter-5.5.tgz'
                    sh 'tar -czvf apache-jmeter-5.5.tgz apache-jmeter-5.5'
                    sh './build.sh'
                    sh 'rm -r apache-jmeter-5.5 && rm -r apache-jmeter-5.5.tgz'
                    sh 'cp ../src/main/resources/perform_test.jmx test'
                    }
                }
            }
        }

        stage ("Run Jmeter Performance Test") {
            steps{
                script {
                    dir('jmeter-docker') {
                        if(fileExists("apache-jmeter-5.5.tgz")){
                            sh 'rm -r apache-jmeter-5.5.tgz'
                        }
                    sh './run.sh -n -t test/perform_test.jmx -l test/perform_test.jtl'
                    sh 'pwd'
                    sh 'docker cp jmeter:/home/jmeter/apache-jmeter-5.5/test/perform_test.jtl $(pwd)/test'
                    perfReport 'test/perform_test.jtl'
                    }
                }
            }
        }

stage ("Generate Taurus Report") {
   steps{
       script {
            dir('jmeter-docker') {
               sh 'pip install bzt'
               sh 'export PATH=$PATH:/home/jenkins/.local/bin'

               BlazeMeterTest: {
                   sh '$(pwd)/bzt test/perform_test.jtl -report'
               }
            }
       }
   }
}








        //stage('NPM build') {
        //    steps {
        //        script {
                    //sh 'mvn install'
                    //sh 'mvn run build'
        //        }
        //    }
        //}

        //stage('SonarQube analysis') {
        //    steps {
        //        withSonarQubeEnv(credentialsId: "sonarqube-server", installationName: "sonarqube-server"){
        //        sh 'mvn run sonar'
        //        }
        //    }
        //}

        
    }
}
