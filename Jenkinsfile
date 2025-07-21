pipeline {
    agent any

    environment {
        PROJECT_NAME = "Universal-SCA-Scan"
        DEP_CHECK_PATH = "/opt/dependency-check/dependency-check/bin/dependency-check.sh"
    }

    stages {

        stage('Clone Repository') {
            steps {
                echo 'Cloning the GitHub Repository.....'
                sh '''
                    rm -rf temp_repo || true
                    git clone --depth=1 https://github.com/Akashsonawane571/devsecops-test.git temp_repo
                '''
            }
        }

        /*
        stage('Secret Scan (TruffleHog)') {
            steps {
                echo 'Running TruffleHog on latest commit only...'
                sh '''
                    cd temp_repo
                    trufflehog --regex --entropy=True --max_depth=10 . > ../trufflehog_report.txt || true
                    cd ..
                    rm -rf temp_repo
                '''
                archiveArtifacts artifacts: 'trufflehog_report.txt', onlyIfSuccessful: false
            }
        }
        */

        stage('Dependency Check (OWASP)') {
            steps {
                echo 'Running OWASP Dependency-Check...'
                sh '''
                    cd temp_repo

                    # Run OWASP Dependency-Check
                    '"${DEP_CHECK_PATH}"' \
                        --project "${PROJECT_NAME}" \
                        --scan . \
                        --format "ALL" \
                        --out ../dependency-check-report || true

                    cd ..
                '''
                archiveArtifacts artifacts: 'dependency-check-report/*', onlyIfSuccessful: false
            }
        }

        /*
        stage('SonarQube Scan') {
            steps {
                echo 'Starting SonarQube SAST Scan...'
                withSonarQubeEnv('sonar') {
                    sh "${tool 'sonar-scanner'}/bin/sonar-scanner"
                }
            }
        }
        */

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image...'
                // Uncomment below if needed
                // sh 'docker build -t juice-shop .'
            }
        }
    }
}
