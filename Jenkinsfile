pipeline {
  agent any
  stages {
    stage('Initial cleanup') {
      steps {
        // Clean the workspace directory
        dir(path: "${WORKSPACE}") {
          deleteDir()
        }
      }
    }

    stage('Checkout SCM') {
      steps {
        // Checkout the code from the main branch
        git(branch: 'main', url: 'https://github.com/AyopoB/php-todo.git')
      }
    }

    stage('Check Composer and PHP') {
      steps {
        sh 'php -v'
        sh 'composer --version'
      }
    }

    stage('Prepare Dependencies') {
      steps {
        script {
          try {
            sh '''
            if [ ! -f .env ]; then
              mv .env.sample .env
            fi
            '''
            sh 'mkdir -p bootstrap/cache'
            sh 'chmod -R 775 bootstrap/cache'
            sh 'composer install --no-dev --optimize-autoloader || { echo "Composer install failed"; exit 1; }'
            sh 'php artisan migrate --force || { echo "Migrate failed"; exit 1; }'
            sh 'php artisan db:seed --force || { echo "Seeding failed"; exit 1; }'
            sh 'php artisan key:generate || { echo "Key generation failed"; exit 1; }'
          } catch (Exception e) {
            currentBuild.result = 'FAILURE'
            throw e
          }
        }
      }
    }
  }
}
