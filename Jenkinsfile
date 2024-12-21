pipeline {
  agent any
  stages {
    stage('Initial cleanup') {
      steps {
        dir(path: "${WORKSPACE}") {
          deleteDir()
        }

      }
    }

    stage('Checkout SCM') {
      steps {
        git(branch: 'main', url: 'https://github.com/AyopoB/php-todo.git')
      }
    }

    stage('Prepare Dependencies') {
      steps {
        sh 'rm -rf vendor'
        sh 'mv .env.sample .env'
        sh 'composer update'
        sh 'composer install'
        sh 'php artisan migrate'
        sh 'php artisan db:seed'
        sh 'php artisan key:generate'
      }
    }

  }
}