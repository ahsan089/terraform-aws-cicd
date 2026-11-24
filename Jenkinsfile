
pipeline {
    agent any

    environment {
        AWS_REGION = 'eu-north-1'
        WEB_INSTANCE_ID = 'i-0227b35933b8e7880'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo 'GitHub checkout successful'
            }
        }

        stage('Build') {
            steps {
                sh '''
                    test -f site/index.html
                    echo "Website files found"
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    grep -q "Deployment Successful" site/index.html
                    echo "Website validation passed"
                '''
            }
        }

        stage('Deploy') {
            steps {
                script {
                    def commandId = sh(
                        script: """
                            aws ssm send-command \
                              --region ${AWS_REGION} \
                              --instance-ids ${WEB_INSTANCE_ID} \
                              --document-name AWS-RunShellScript \
                              --parameters 'commands=["mkdir -p /var/backups/nginx","if [ -f /var/www/html/index.nginx-debian.html ]; then cp -n /var/www/html/index.nginx-debian.html /var/backups/nginx/; fi","cp /var/www/html/index.html /var/www/html/index.html.bak 2>/dev/null || true","test -f /var/www/html/index.html.bak || true"]' \
                              --query 'Command.CommandId' \
                              --output text
                        """,
                        returnStdout: true
                    ).trim()

                    echo "SSM Command ID: ${commandId}"
                }
            }
        }
    }
}
