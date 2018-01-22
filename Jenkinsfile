pipeline {
  agent none

  environment {
    MAJOR_VERSION = 1
  }


  stages {
    stage ('Unit Tests') {
      agent {
        label 'apache'
      }
      steps {
        sh 'ant -f test.xml -v'
        junit 'reports/result.xml'
      }
    }
    stage ('build') {
      agent {
        label 'apache'
      }
      steps {
        sh 'ant -f build.xml -v'
      }
      post {
        success {
          archiveArtifacts artifacts: 'dist/*.jar', fingerprint: true
        }
      }
    }
    stage ('deploy') {
      agent {
        label 'apache'
      }
      steps {
        sh "if ![ -d '/var/www/html/rectangles/all/${env.BRANCH_NAME}' ]; then mkdir /var/www/html/rectangles/all/${env.BRANCH_NAME}; fi"
        sh "cp dist/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/all/${env.BRANCH_NAME}/"
      }
    }
    stage ("Running on CentOS") {
      agent {
        label 'CentOS'
      }
      steps {
        sh "wget http://172.28.128.20:8081/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
      }
    }
    stage ("Test on Debian") {
      agent {
        docker 'openjdk:8u151-jre-alpine'
      }
      steps {
        sh "wget http://172.28.128.20:8081/rectangles/all/${env.BRANCH_NAME}/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
        sh "java -jar rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar 3 4"
      }
    }
    stage ("Promote to Green") {
      agent {
        label 'apache'
      }
      when {
        branch 'master'
      }
      steps {
        sh "cp /var/www/html/rectangles/all/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar /var/www/html/rectangles/green/rectangle_${env.MAJOR_VERSION}.${env.BUILD_NUMBER}.jar"
      }
    }
    stage ("Promote developement Branch to Master") {
      agent {
        label 'apache'
      }
      when {
        branch 'developement'
      }
      steps {
        echo "Stashing Any Local Changes."
        sh 'git stash'
        echo "Checking Out Developement Branch"
        sh 'git checkout developement'
        echo 'Checking Out Master Branch'
        sh 'git pull origin'
        sh 'git checkout master'
        echo 'Merging Developement into Master Branch'
        sh 'git merge developement'
        echo 'Pushing to Orign Master'
        sh 'git push origin master'
        echo 'Tagging the Release'
        sh "git tag rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
        sh "git push origin rectangle-${env.MAJOR_VERSION}.${env.BUILD_NUMBER}"
      }
    }
  }


}
