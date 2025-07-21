pipeline {
    agent any

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning the GitHub Repository.....'
                sh '''
                  rm -rf temp_repo
                  git clone --depth=1 https://github.com/Akashsonawane571/devsecops-test.git temp_repo
                '''
            }
        }

        /*
        stage('Secret Scan (TruffleHog)') {
            steps {
                echo 'Running TruffleHog on latest commit only...'
                sh '''
                  git clone --depth=1 https://github.com/Akashsonawane571/devsecops-test.git temp_repo
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
                  /opt/dependency-check/dependency-check/bin/dependency-check.sh --project "Universal-SCA-Scan" --scan . --format ALL --out ../dependency-check-report || true
                  cd ..
                '''
                sh 'mkdir -p dependency-check-report' // makes sure directory exists even if empty
                archiveArtifacts artifacts: 'dependency-check-report/*', onlyIfSuccessful: false
            }
        }

        /*
        stage('SonarQube Scan') {
            steps {
                echo 'Starting SonarQube SAST Scan...'
                withSonarQubeEnv('sonar') {
                    sh "${SONARQUBE_SCANNER_HOME}/bin/sonar-scanner"
                }
            }
        }
        */

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image...'
                // sh 'docker build -t juice-shop .'
            }
        }
    }
}
