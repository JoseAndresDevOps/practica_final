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
            }
        }

        stage("Publish to Nexus") {
            steps {
                script {
                    // Read POM xml file using 'readMavenPom' step , this step 'readMavenPom' is included in: https://plugins.jenkins.io/pipeline-utility-steps
                    pom = readMavenPom file: "pom.xml"
                    // Find built artifact under target folder
                    filesByGlob = findFiles(glob: "target/*.${pom.packaging}")
                    // Print some info from the artifact found
                    echo "${filesByGlob[0].name} ${filesByGlob[0].path} ${filesByGlob[0].directory} ${filesByGlob[0].length} ${filesByGlob[0].lastModified}"
                    // Extract the path from the File found
                    artifactPath = filesByGlob[0].path
                    // Assign to a boolean response verifying If the artifact name exists
                    artifactExists = fileExists artifactPath                    
                    if(artifactExists) {
                        echo "*** File: ${artifactPath}, group: ${pom.groupId}, packaging: ${pom.packaging}, version ${pom.version}"
                        versionPom = "${pom.version}"
			} else {
                        error "*** File: ${artifactPath}, could not be found"
                    }
                }
            }
        }


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
