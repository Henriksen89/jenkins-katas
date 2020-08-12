pipeline {
  agent any
  environment {
    docker_username = 'henriksen89'
  }
  stages {
    stage('Clone down') {
      steps {
        stash excludes: '.git/', name: 'code'
      }
    }
    stage('Paralel execution') {
      parallel {
        stage('Say Hello') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('Build app') {
          options {
            skipDefaultCheckout()
          }

          agent {
            docker {
              image 'gradle:jdk11'
            }
          }

          steps {
            unstash 'code'
            sh 'ls'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            stash excludes: '.git/', name: 'code'
            deleteDir()
            sh 'ls'
          }
        }

        stage('Test app') {
          options {
            skipDefaultCheckout()
          }
          agent {
            docker {
              image 'gradle:jdk11'
            }
          }
          steps {
            unstash 'code'
            sh 'ci/unit-test-app.sh'
            junit 'app/build/test-results/test/TEST-*.xml'
          }
        }
      }
    }

    stage('push docker app') {
      environment {
        DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
      }
        options {
            skipDefaultCheckout()
        }
      steps {
        unstash 'code' //unstash the repository code
        sh 'ci/build-docker.sh'
        pushIfMaster()
      }
    }

    stage('component-test') {
      when {
        options {
          skipDefaultCheckout(true)
        }
      }
      steps{
        unstash 'code'
        sh 'ci/component-test.sh'
      }
    }
  }
  void pushIfMaster() {
    if (BRANCH_NAME=="master"){
      sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin'
      sh 'ci/push-docker.sh'
    }
  }
}