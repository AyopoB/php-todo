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
        git(branch: 'main', url: 'https://github.com/gashawgedef/php-todo.git')
      }
    }
    stage('Check DB Connection') {
      steps {
        script {
          sh 'mysql -h ${DB_HOST} -u ${DB_USERNAME} -p${DB_PASSWORD} -e "SELECT 1;"'
        }
      }
    }


    stage('Prepare Dependencies') {
      steps {
        sh 'mv .env.sample .env'
        sh 'composer install'
        sh 'php artisan migrate'
        sh 'php artisan db:seed'
        sh 'php artisan key:generate'
      }
    }

  }
}