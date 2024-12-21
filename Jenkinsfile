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

    stage('Prepare Dependencies') {
      steps {
        // Set up Laravel environment and dependencies
        sh '''
        if [ ! -f .env ]; then
          mv .env.sample .env
        fi
        '''
        sh 'mkdir -p bootstrap/cache'
        sh 'chmod -R 775 bootstrap/cache'
        sh 'composer install --no-dev --optimize-autoloader'
        sh 'php artisan migrate --force'
        sh 'php artisan db:seed --force'
        sh 'php artisan key:generate'
      }
    }
  }
}
