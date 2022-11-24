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
        stage('build') {
            steps {
                sh 'mvn --version'
                sh "mvn clean package -DskipTest"
            }
        }
    }
}
