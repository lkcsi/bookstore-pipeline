pipeline {
    agent any
    environment {
        AUTH_SECRET=credentials('auth-secret')
        API_KEY=credentials('api-key')
        BOOKS_REPOSITORY='SQL'
        BOOKS_DB_HOST='books-db-1'
        BOOKS_DB_PASSWORD=credentials('books-db-password')
        BOOKS_DB_PORT=3306
        BOOKS_API_PORT=8081
        BOOKS_FRONT_PORT=8082
    }
    stages {
        stage("build") {
            steps {
                echo 'build'
                script {
                    try {
                        sh 'docker stop $(docker ps -a -q)'
                        sh 'docker rm $(docker ps -a -q)'
                    } catch (err) {
                        echo err
                    }                    
                }
                dir('api') {
                    git branch: 'main', url: 'https://github.com/lkcsi/bookstore.git'
                    script {
                        sh 'docker compose build'
                        sh 'docker compose up -d'
                    }
                }
                dir('front') {
                    git branch: 'main', url: 'https://github.com/lkcsi/bookstore-front.git'
                    script {
                        sh 'docker compose build'
                        sh 'docker compose up -d'
                    }
                }
            }
        }
        stage("wait") {
            steps{
                sleep(time:20,unit:"SECONDS")
            }
            
        }
        stage("api-test") {
            environment {
                BOOK_USER='test'
                PASSWORD='password'
                ENDPOINT='http://localhost:8081'
            }
            steps {
                dir('tests') {
                    git branch: 'main', url: 'https://github.com/lkcsi/bookstore-api-test.git'
                    sh 'virtualenv .venv'
                    sh '. .venv/bin/activate'
                    sh 'pip install -r requirements.txt'
                    sh 'pytest -k "test"'
                }
            }
        }
    }
}
