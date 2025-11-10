pipeline {
		agent any
		options {
				timestamps()
				ansiColor('xterm')
				skipStagesAfterUnstable()
		}
		// Try to run automatically on changes; adjust or replace with webhook-triggered job if preferred
		triggers {
				// Poll SCM every 5 minutes as a fallback for auto-deploy on changes
				cron('H/5 * * * *')
		}
		stages {
				stage('Checkout') {
						steps {
								// Standard git checkout
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
	# Use Docker Compose V2 (docker compose) if available, otherwise fall back to docker-compose
	if command -v docker-compose >/dev/null 2>&1; then
		echo "Using docker-compose"
		docker-compose pull || true
		docker-compose up -d --build
	else
		echo "Using docker compose (v2)"
		docker compose pull || true
		docker compose up -d --build
	fi
else
	echo "No docker-compose file found in workspace - skipping deploy"
fi
'''
										} else {
												// Windows agent - run via PowerShell to better support pipelines
												bat '''
powershell -NoProfile -Command "\
if (Test-Path -Path 'docker-compose.yaml' -PathType Leaf -or Test-Path -Path 'docker-compose.yml' -PathType Leaf) { \
	Write-Host 'Detected Windows agent'; \
	if (Get-Command docker-compose -ErrorAction SilentlyContinue) { \
		docker-compose pull | Out-Null; \
		docker-compose up -d --build; \
	} else { \
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
						echo 'Deployment pipeline finished: SUCCESS'
				}
				failure {
						echo 'Deployment pipeline finished: FAILURE'
				}
				unstable {
						echo 'Deployment pipeline finished: UNSTABLE'
				}
		}
}

