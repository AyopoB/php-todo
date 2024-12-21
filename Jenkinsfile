pipeline {
    agent any
    stages {
        stage("Initial Cleanup") {
            steps {
                dir("${WORKSPACE}") {
                    deleteDir()
                }
            }
        }
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/AyopoB/php-todo.git'
            }
        }
        stage('Prepare Dependencies') {
            steps {
                sh 'mkdir -p bootstrap/cache'
                sh 'chmod -R 775 bootstrap/cache'
                sh 'mv .env.sample .env'
                sh 'composer install'
                sh 'php artisan migrate'
                sh 'php artisan db:seed'
                sh 'php artisan key:generate'
            }
        }
        stage('Execute Unit Tests') {
            steps {
                sh './vendor/bin/phpunit'
            }
        }
    }
}