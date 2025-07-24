pipeline {
    agent any

    stages {

        /*
        // === Commented Out: Secret Scan (TruffleHog) ===
        stage('Secret Scan (TruffleHog)') {
            steps {
                echo 'Skipping TruffleHog secret scan...'
            }
        }

        // === Commented Out: Dependency Check ===
        stage('Dependency Check') {
            steps {
                echo 'Skipping Dependency Check...'
            }
        }

        // === Commented Out: SonarQube Scan ===
        stage('SonarQube Scan') {
            steps {
                echo 'Skipping SonarQube scan...'
            }
        }

        // === Commented Out: Build Project ===
        stage('Build Project') {
            steps {
                echo 'Skipping Maven build...'
            }
        }

        // === Commented Out: Build Docker Image ===
        stage('Build Docker Image') {
            steps {
                echo 'Skipping Docker build...'
            }
        }

        // === Commented Out: Deploy to Server ===
        stage('Deploy to Server') {
            steps {
                echo 'Skipping deployment...'
            }
        }
        */

        // === ✅ Active Stage: OWASP ZAP DAST Scan ===
        stage('OWASP ZAP DAST Scan') {
            steps {
                script {
                    echo "🔍 Starting OWASP ZAP scan on http://192.168.18.137:3000"

                    // Pull and run the ZAP scan
                    sh '''
                        docker run --rm -v $WORKSPACE:/zap/wrk/:rw owasp/zap2docker-stable zap-baseline.py \
                            -t http://192.168.18.137:3000 \
                            -r zap_report.html \
                            -x zap_report.xml \
                            -J zap_report.json || true
                    '''
                }

                // Archive all ZAP reports
                archiveArtifacts artifacts: 'zap_report.*', allowEmptyArchive: true
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning up temporary scan files..."
            sh 'rm -f zap_report.*'
        }
    }
}
