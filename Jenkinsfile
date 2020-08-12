pipeline {
  agent any
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
	          sh "ls"
            deleteDir()
	          sh "ls"
          }
        }
      }
    }
  }
}
