
pipeline {
    agent none

    options {
        timestamps()
        disableConcurrentBuilds()
        buildDiscarder(logRotator(
            numToKeepStr: '10'
        ))
        timeout(time: 20, unit: 'MINUTES')
    }

    environment {
        REPO_URL = 'https://github.com/Najil-kk/My-details.git'
        APP_DIR = 'My-details'
        DEPLOY_DIR = '/var/www/html'
        BACKUP_DIR = '/var/backups/website'
    }

    stages {

        // 1. Checkout Source Code
        stage('Checkout') {
            agent {
                label 'buildnode'
            }

            steps {
                deleteDir()

                echo 'Cloning source code...'

                git branch: 'main',
                    url: "${REPO_URL}"

                sh '''
                    echo "Checking repository..."
                    ls -la

                    echo "Checking website files..."
                    ls -la ${APP_DIR}
                '''

                stash name: 'webpage',
                      includes: 'My-details/**',
                      useDefaultExcludes: false
            }
        }

        // 2. Validate Website
        stage('Validation') {
            agent {
                label 'buildnode'
            }

            steps {
                unstash 'webpage'

                script {
                    if (!fileExists('My-details/index.html')) {
                        error "index.html not found!"
                    }
                }

                sh '''
                    echo "Website validation successful"

                    test -s My-details/index.html

                    echo "HTML file exists and is not empty"
                '''
            }
        }

        // 3. Deployment
        stage('Deploy') {
            agent {
                label 'deploynode'
            }

            steps {
                deleteDir()

                unstash 'webpage'

                sh '''
                    set -e

                    echo "Starting deployment..."

                    # Create backup directory
                    sudo mkdir -p ${BACKUP_DIR}

                    # Backup existing website
                    if [ -d "${DEPLOY_DIR}" ] &&
                       [ "$(ls -A ${DEPLOY_DIR})" ]; then

                        sudo tar -czf \
                        ${BACKUP_DIR}/backup-${BUILD_NUMBER}.tar.gz \
                        -C ${DEPLOY_DIR} .

                    fi

                    # Create temporary deployment directory
                    sudo mkdir -p ${DEPLOY_DIR}

                    # Copy new website files
                    sudo cp -r My-details/. ${DEPLOY_DIR}/

                    # Set permissions
                    sudo chown -R www-data:www-data ${DEPLOY_DIR}

                    sudo find ${DEPLOY_DIR} -type d -exec chmod 755 {} \\;

                    sudo find ${DEPLOY_DIR} -type f -exec chmod 644 {} \\;

                    echo "Deployment completed!"
                '''
            }
        }

        // 4. Verify Deployment
        stage('Verify') {
            agent {
                label 'deploynode'
            }

            steps {
                sh '''
                    set -e

                    echo "Verifying deployed website..."

                    test -s ${DEPLOY_DIR}/index.html

                    echo "Deployment verification successful"
                '''
            }
        }
    }

    // 5. Post Build Actions
    post {

        success {
            echo '''
            =================================
              PIPELINE SUCCESSFUL
              Website deployed successfully
            =================================
            '''
        }

        failure {
            echo '''
            =================================
              PIPELINE FAILED
              Please check Jenkins console
            =================================
            '''
        }

        always {
            echo "Pipeline execution finished."
        }
    }
}
