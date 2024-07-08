version = 'none'

pipeline {
  options {
    buildDiscarder(logRotator(numToKeepStr: '3'))
  }
  agent any
  stages {
    stage('Build') {
      steps {
        sh "poetry15 config virtualenvs.in-project true"
        sh "poetry15 config --list"
        sh "poetry15 install"
      }
    }

    stage('Publish Development Version') {
      when {
        not { branch 'main' }
      }
      steps {
        script {
          def dateFormat = new java.text.SimpleDateFormat("yyyyMMddHHmm")
          def date = new java.util.Date()

          DATE = dateFormat.format(date)

          sh "poetry15 version prepatch"

          version = sh(script: "poetry15 version | awk  '{print \$2}'", returnStdout: true).trim().replaceAll("a0\$", "a$DATE")

          sh "poetry15 version $version"

          withCredentials([usernamePassword(credentialsId:'DOCKER_REPOSITORY',
                                            usernameVariable: 'USERNAME',
                                            passwordVariable: 'PASSWORD')]){
            sh "poetry15 config repositories.datalab-dev https://pypi.datalabserasaexperian.com.br/repository/pypi-develop/"
            sh "poetry15 publish --build -r datalab-dev --username ${USERNAME} --password ${PASSWORD}"
          }
        }
      }
    }

    stage('Bumping Version to Release') {
      when {
        branch 'main'
      }
      steps {
        script {
          sh "git checkout main"
          sh "poetry15 version patch"

          version = sh(script: "poetry15 version | awk  '{print \$2}'", returnStdout: true).trim()

          sh "git add ."
          sh "git commit -m '[Jenkins] New Release $version'"
          sh "git tag $version"
        }
      }
    }

    stage('Publish Release Version') {
      when {
        branch 'main'
      }
      steps {
        script {
          sh "git checkout main"

          withCredentials([usernamePassword(credentialsId:'DOCKER_REPOSITORY',
                  usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]){
            sh "poetry15 config repositories.datalab https://pypi.datalabserasaexperian.com.br/repository/pypi-hosted/"
            sh "poetry15 publish --build -r datalab --username ${USERNAME} --password ${PASSWORD}"
          }
        }
      }
    }

    stage('Pushing everything') {
      when {
        branch 'main'
      }
      steps {
        script {
          sh "poetry15 version prepatch"
          sh "git add ."
          sh "git commit -m '[Jenkins] Start Pre-Release'"

          sh "git push origin main --tags"
        }
      }
    }
  }
}


