pipeline {
  agent any
  environment {
        CC = 'clang'
  }
  stages {
      stage('Example') {
          environment {
        DEBUG_FLAGS = '-g'
          }
          steps {
        sh 'printenv'
          }
      }
  }

  stages {
    stage('Clone Down') {
      steps {
        stash excludes: '.git', name: 'code'
      }
    }

    stage('Parallel execution') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('Build app') {
          options {
            skipDefaultCheckout true
          }

          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
              sh 'ls'
            deleteDir()
              sh 'ls'
          }
        }
      }
    }

      stage('push docker') {
          environment {
          DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
          }
        steps {
          unstash 'code' //unstash the repository code
          sh 'ci/build-docker.sh'
          sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
          sh 'ci/push-docker.sh'
        }
      }
  }
}
