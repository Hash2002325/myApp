pipeline {
    agent any

    options {
        timestamps()
        skipStagesAfterUnstable()
    }

    // Automatically run every 5 minutes (optional — remove if you use webhooks)
    triggers {
        cron('H/5 * * * *')
    }

    stages {
        stage('Checkout') {
            steps {
                // Pull the latest code from your GitHub repo
                checkout scm
            }
        }

        stage('Update services') {
            steps {
                script {
                    if (isUnix()) {
                        sh '''
set -e
echo "Detected Unix agent"

if [ -f docker-compose.yaml ] || [ -f docker-compose.yml ]; then
    echo "Running docker-compose deployment..."

    # Clean up old containers to avoid name conflicts
    echo "Cleaning up old containers..."
    if command -v docker-compose >/dev/null 2>&1; then
        echo "Using docker-compose (v1)"
        docker rm -f backend_c frontend_c mongodb_c 2>/dev/null || true
        docker-compose down --remove-orphans || true
        docker-compose pull || true
        docker-compose up -d --build
    else
        echo "Using docker compose (v2)"
        docker rm -f backend_c frontend_c mongodb_c 2>/dev/null || true
        docker compose down --remove-orphans || true
        docker compose pull || true
        docker compose up -d --build
    fi
else
    echo "No docker-compose file found in workspace - skipping deploy"
fi
'''
                    } else {
                        bat '''
powershell -NoProfile -Command "\
if (Test-Path -Path 'docker-compose.yaml' -PathType Leaf -or Test-Path -Path 'docker-compose.yml' -PathType Leaf) { \
    Write-Host 'Detected Windows agent'; \
    Write-Host 'Cleaning up old containers...'; \
    if (Get-Command docker-compose -ErrorAction SilentlyContinue) { \
        docker-compose down --remove-orphans; \
        docker-compose pull | Out-Null; \
        docker-compose up -d --build; \
    } else { \
        docker compose down --remove-orphans; \
        docker compose pull | Out-Null; \
        docker compose up -d --build; \
    } \
} else { Write-Host 'No docker-compose file found in workspace - skipping deploy' }"
'''
                    }
                }
            }
        }
    }

    post {
        success {
            echo '✅ Deployment pipeline finished: SUCCESS'
        }
        failure {
            echo '❌ Deployment pipeline finished: FAILURE'
        }
        unstable {
            echo '⚠️ Deployment pipeline finished: UNSTABLE'
        }
    }
}
