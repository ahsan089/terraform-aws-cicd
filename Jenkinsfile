
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
                sh 'test -f site/index.html'
                echo 'Website files found'
            }
        }

        stage('Test') {
            steps {
                sh 'grep -q "Deployment Successful" site/index.html'
                echo 'Website validation passed'
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                python3 - <<'PY'
import os
import json
import base64
import subprocess
import time

region = os.environ["AWS_REGION"]
instance = os.environ["WEB_INSTANCE_ID"]

with open("site/index.html", "rb") as f:
    content = base64.b64encode(f.read()).decode()

commands = [
    "set -e",
    "mkdir -p /var/backups/nginx",
    "if [ -f /var/www/html/index.nginx-debian.html ]; then cp -n /var/www/html/index.nginx-debian.html /var/backups/nginx/; fi",
    "if [ -f /var/www/html/index.html ]; then cp -n /var/www/html/index.html /var/backups/nginx/index.html; fi",
    "printf '%s' '" + content + "' | base64 -d > /tmp/index.html.new",
    "install -o www-data -g www-data -m 0644 /tmp/index.html.new /var/www/html/index.html",
    "nginx -t",
    "systemctl reload nginx"
]

payload = {
    "DocumentName": "AWS-RunShellScript",
    "InstanceIds": [instance],
    "Parameters": {"commands": commands}
}

result = subprocess.run([
    "aws", "ssm", "send-command",
    "--region", region,
    "--cli-input-json", json.dumps(payload),
    "--query", "Command.CommandId",
    "--output", "text"
], capture_output=True, text=True, check=True)

command_id = result.stdout.strip()
print("SSM Command ID:", command_id)

for attempt in range(60):
    check = subprocess.run([
        "aws", "ssm", "get-command-invocation",
        "--region", region,
        "--command-id", command_id,
        "--instance-id", instance,
        "--output", "json"
    ], capture_output=True, text=True)

    if check.returncode == 0:
        response = json.loads(check.stdout)
        status = response["Status"]

        if status == "Success":
            print(response.get("StandardOutputContent", ""))
            print("Deployment successful!")
            break

        if status in ["Failed", "Cancelled", "TimedOut"]:
            print(response.get("StandardErrorContent", ""))
            raise SystemExit("Deployment failed: " + status)

    time.sleep(5)
else:
    raise SystemExit("Timed out waiting for SSM deployment")
PY
                '''
            }
        }
    }
}
