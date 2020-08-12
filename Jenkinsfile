pipeline {
  agent any
  environment{
    docker_username = 'oskardock'
  }
  stages {
    stage('clone down') {
      steps {
        stash(name: 'code', excludes: '.git')
      }
    }

    stage('Build') {
      parallel {
        stage('Parallel execution') {
          steps {
            sh 'echo "hello world"'
          }
        }

        stage('Build app') {
          agent {
            docker {
              image 'gradle:jdk11'
            }

          }
          steps {
            unstash 'code'
            sh 'ci/build-app.sh'
            archiveArtifacts 'app/build/libs/'
            stash(name: 'build')
            sh 'ls'
            deleteDir()
            sh 'ls'
            skipDefaultCheckout true
          }
        }

        stage('Test app') {
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
    stage('Push docker app'){
      environment {
        DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
      }
      when { branch "master" }
      steps{
        unstash 'build'
        sh 'ci/build-docker.sh'
        sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
        sh 'ci/push-docker.sh'
        sh 'echo "Test github webhooks"'
      }
    }
    stage('Component test'){
      when {
        not{
          branch "dev/*" 
        }
      }
      steps {
        unstash 'build'
        sh 'ci/component-test.sh'
      }
    }
  }
}