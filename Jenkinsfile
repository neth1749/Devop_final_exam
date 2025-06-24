pipeline {
    agent any

    triggers {
        pollSCM('H/5 * * * *')  // Poll Git every 5 minutes
    }

    environment {
        ANSIBLE_PLAYBOOK = './web-pod-tasks.yml'
        EMAIL_CC = 'srengty@gmail.com'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'composer install --no-interaction --prefer-dist'
                sh 'npm install'
                sh 'npm run build'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'php artisan migrate --database=sqlite_testing'
                sh 'vendor/bin/phpunit'
            }
        }

        stage('Deploy') {
            steps {
                sh "ansible-playbook ${ANSIBLE_PLAYBOOK}"
            }
        }
    }

    post {
        success {
            echo 'Build and deployment succeeded.'
        }

        failure {
            script {
                // Get commit author email for notification
                def commitAuthor = sh(script: "git log -1 --pretty=format:'%ae'", returnStdout: true).trim()
                emailext(
                    subject: "Build Failed in ${env.JOB_NAME}",
                    body: "The build failed. Please check the Jenkins console output.",
                    to: "${commitAuthor}",
                    cc: "${EMAIL_CC}"
                )
            }
        }
    }
}
