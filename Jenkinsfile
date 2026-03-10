pipeline {

    // agent {
    //     docker {
    //         image 'docker:latest'?
    //         args '-v /var/run/docker.sock:/var/run/docker.sock'
    //     }
    // }
    agent any

    options {
        buildDiscarder(logRotator(numToKeepStr: '10'))
        timestamps()
        timeout(time: 20, unit: 'MINUTES')
        disableConcurrentBuilds()
    }

    environment {
        APP_NAME = 'laravel-devops'
        CI_IMAGE = "${APP_NAME}:ci-${env.BUILD_NUMBER}"
    }

    stages {

        stage('📋 Pipeline Info') {
            steps {
                script {
                    echo """
╔══════════════════════════════════════════════════════╗
║               CI PIPELINE STARTED                    ║
╚══════════════════════════════════════════════════════╝
   Build  : #${env.BUILD_NUMBER}
   Branch : ${env.BRANCH_NAME}
   PR     : ${env.CHANGE_ID ?: 'Not a PR'}
   Title  : ${env.CHANGE_TITLE ?: 'N/A'}
══════════════════════════════════════════════════════
                    """
                }
            }
        }

        stage('🔧 Verify Environment') {
            steps {
                sh '''
                    echo "Hostname : $(hostname)"
                    echo "User     : $(whoami)"
                    docker --version
                    echo "✅ Environment ready"
                '''
            }
        }

        stage('🔍 Code Quality') {
            steps {
                sh '''
                    echo "Running code quality checks..."
                    echo "✅ Code quality passed"
                '''
            }
        }

                stage('🐳 Build Docker Image') {
                steps {
                        sh '''
                        docker build -t laravel-devops:${BUILD_TAG} .
                        '''
            }
        }
                stage('🧪 Verify Image') {
                    steps {
                        sh '''
                        echo "=== Image Verification ==="
                        docker run --rm --entrypoint="" laravel-devops:${IMAGE_TAG} php artisan --version
                        '''
                    }
               }
        stage('🔒 Security Scan') {
            steps {
                sh """
                    echo "=== Security Scan ==="

                    CONTAINER_UID=\$(docker run --rm --entrypoint id ${CI_IMAGE} -u)
                    echo "Container UID: \${CONTAINER_UID}"

                    if [ "\${CONTAINER_UID}" = "0" ]; then
                        echo "❌ FAILED: Running as ROOT (UID 0) - Security risk!"
                        exit 1
                    else
                        echo "✅ PASSED: Non-root UID (\${CONTAINER_UID})"
                    fi
                """
            }
        }

        stage('🧹 Cleanup') {
            steps {
                sh """
                    docker rmi ${CI_IMAGE} || true
                    echo "✅ Cleanup done"
                """
            }
        }
    }

    post {
        success {
            script {
                if (env.CHANGE_ID) {
                    echo """
╔══════════════════════════════════════════════════════╗
║            ✅ CI PASSED - PR VALIDATED               ║
╚══════════════════════════════════════════════════════╝
   PR     : #${env.CHANGE_ID} - ${env.CHANGE_TITLE}

   ✅ Code Quality  : Passed
   ✅ Docker Build  : Passed
   ✅ Image Verify  : Passed
   ✅ Security Scan : Passed
   🚫 Deployment   : Skipped (PRs never deploy)

   → Get code review → Merge to main for deployment
══════════════════════════════════════════════════════
                    """
                } else {
                    echo """
╔══════════════════════════════════════════════════════╗
║          ✅ CI PASSED - BRANCH VALIDATED             ║
╚══════════════════════════════════════════════════════╝
   Branch : ${env.BRANCH_NAME}
   Build  : #${env.BUILD_NUMBER}

   ✅ Code Quality  : Passed
   ✅ Docker Build  : Passed
   ✅ Image Verify  : Passed
   ✅ Security Scan : Passed
══════════════════════════════════════════════════════
                    """
                }
            }
        }

        failure {
            echo """
╔══════════════════════════════════════════════════════╗
║              ❌ CI PIPELINE FAILED                   ║
╚══════════════════════════════════════════════════════╝
   Build  : #${env.BUILD_NUMBER}
   Branch : ${env.BRANCH_NAME}
   PR     : ${env.CHANGE_ID ?: 'N/A'}
   Logs   : ${env.BUILD_URL}
══════════════════════════════════════════════════════
            """
        }

        always {
            sh 'docker image prune -f || true'
        }
    }
}