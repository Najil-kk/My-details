
pipeline {
    agent none

    stages {

        stage('Build') {
            agent {
                label 'buildnode'
            }

            steps {
                deleteDir()

                git branch: 'main',
                    url: 'https://github.com/Najil-kk/My-details.git'

                sh '''
                    echo "Build Started"

                    ls -la

                    test -f index.html

                    echo "Build Completed"
                '''

                stash name: 'webpage',
                      includes: '**',
                      excludes: '.git/**'
            }
        }

        stage('Deploy') {
            agent {
                label 'deploynode'
            }

            steps {
                deleteDir()

                unstash 'webpage'

                sh '''
                    echo "Deployment Started"

                    sudo mkdir -p /var/www/html

                    sudo rm -rf /var/www/html/*

                    sudo cp -r . /var/www/html/

                    sudo chown -R www-data:www-data /var/www/html

                    echo "Deployment Completed"

                    ls -la /var/www/html/
                '''
            }
        }
    }

    post {
        success {
            echo 'Website deployed successfully!'
        }

        failure {
            echo 'Deployment failed!'
        }
    }
}
