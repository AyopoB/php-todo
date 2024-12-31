pipeline {
    agent any
    environment {
        scannerHome = tool 'SonarQubeScanner'
        JAVA_HOME = '/usr/lib/jvm/java-17-openjdk-amd64'
        PATH = "${JAVA_HOME}/bin:${PATH}"
        SONAR_HOST_URL = 'http://54.83.16.224:9000'
    }
    stages {
        stage('Initial Cleanup') {
            steps {
                deleteDir()
            }
        }
        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/AyopoB/php-todo.git'
            }
        }
        stage('Prepare Dependencies') {
            steps {
                sh '''
                    mv .env.sample .env
                    mkdir -p bootstrap/cache storage/framework/{sessions,views,cache}
                    chown -R jenkins:jenkins bootstrap storage
                    chmod -R 775 bootstrap storage
                    composer install
                    php artisan migrate
                    php artisan db:seed
                    php artisan key:generate
                '''
            }
        }
        stage('Execute Unit Tests') {
            steps {
                sh './vendor/bin/phpunit'
            }
        }
        stage('Code Analysis') {
            steps {
                sh 'phploc app/ --log-csv build/logs/phploc.csv'
            }
        }
        stage('Plot Code Coverage Report') {
            steps {
                script {
                    def plots = [
                        ['A - Lines of code', 'Lines of Code', 'phploc.csv'],
                        ['B - Structures Containers', 'Count', 'phploc.csv']
                        // Add other plots as needed
                    ]
                    plots.each { plot ->
                        plot csvFileName: "plot-${UUID.randomUUID()}.csv",
                             csvSeries: [[displayTableFlag: false, file: "build/logs/${plot[2]}"]],
                             group: 'phploc',
                             numBuilds: '100',
                             style: 'line',
                             title: plot[0],
                             yaxis: plot[1]
                    }
                }
            }
        }
        stage('Package Artifact') {
            steps {
                sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
            }
        }
        stage('Upload Artifact to Artifactory') {
            steps {
                script {
                    def server = Artifactory.server 'artifactory-server'
                    def uploadSpec = """{
                        "files": [{
                            "pattern": "php-todo.zip",
                            "target": "todo-dev-local/php-todo",
                            "props": "type=zip;status=ready"
                        }]
                    }"""
                    server.upload spec: uploadSpec
                }
            }
        }
        stage('Deploy to Dev Environment') {
            steps {
                build job: 'ansible-config-mgt/main',
                      parameters: [string(name: 'env', value: 'dev')],
                      propagate: false,
                      wait: true
            }
        }
        stage('SonarQube Quality Gate') {
            when {
                branch pattern: '^develop*|^hotfix*|^release*|^main*', comparator: 'REGEXP'
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    withCredentials([usernamePassword(credentialsId: 'sonarqube', usernameVariable: 'SONAR_USER', passwordVariable: 'SONAR_TOKEN')]) {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=php-todo \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONAR_HOST_URL} \
                            -Dsonar.login=$SONAR_TOKEN
                        """
                    }
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline completed!'
        }
        failure {
            echo 'Pipeline failed!'
        }
    }
}
