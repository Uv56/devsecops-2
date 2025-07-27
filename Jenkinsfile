pipeline {
    agent any

    environment {
        DEPENDENCY_CHECK = '/opt/dependency-check/dependency-check/bin/dependency-check.sh'
        ZAP_REPORT_HTML = 'zap_report.html'
        ZAP_REPORT_XML  = 'zap_report.xml'
        ZAP_REPORT_JSON = 'zap_report.json'
        TARGET_URL      = 'http://localhost:3000' // Replace with actual target
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning the GitHub Repository...'
                sh '''
                    rm -rf temp_repo
                    git clone --depth=1 https://github.com/Akashsonawane571/devsecops-test.git temp_repo
                '''
            }
        }

        stage('Secret Scan (TruffleHog)') {
            steps {
                echo 'Running TruffleHog on latest commit...'
                sh '''
                    docker run --rm -v $(pwd)/temp_repo:/project trufflesecurity/trufflehog \
                    filesystem /project > trufflehog_report.txt || true
                '''
                archiveArtifacts artifacts: 'trufflehog_report.txt', onlyIfSuccessful: false
            }
        }

      stage('Dependency Check (OWASP)') {
    steps {
        echo 'Running OWASP Dependency-Check in Docker...'
        withCredentials([string(credentialsId: 'NVD_API_KEY', variable: 'NVD_API_KEY')]) {
            sh '''
                mkdir -p dependency-check-report
                docker run --rm \
                  -v $(pwd)/dependency-check-report:/report \
                  -v $(pwd)/temp_repo:/src \
                  -e NVD_API_KEY=$NVD_API_KEY \
                  owasp/dependency-check:latest \
                  --scan /src \
                  --format "ALL" \
                  --project "Universal-SCA-Scan" \
                  --out /report
            '''
        }
        archiveArtifacts artifacts: 'dependency-check-report/*', onlyIfSuccessful: false
    }
}

        stage('SonarQube Scan') {
          steps {
        echo 'Starting SonarQube SAST Scan...'
        withSonarQubeEnv('sonarqube') {
            withCredentials([string(credentialsId: 'newtoken', variable: 'SONAR_TOKEN')]) {
                sh '''
                    docker run --rm \
                      -v "$PWD/temp_repo:/usr/src" \
                      sonarsource/sonar-scanner-cli \
                      -Dsonar.projectKey=devsecops-test \
                      -Dsonar.sources=. \
                      -Dsonar.host.url=http://192.168.18.137:9000 \
                      -Dsonar.login=$SONAR_TOKEN
                '''
            }
        }
    }
}

        stage('Build Project') {
            steps {
                echo 'Building the Java project with Maven...'
                dir('temp_repo') {
                    sh 'mvn clean install -DskipTests'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker Image...'
                dir('temp_repo') {
                    sh 'docker build -t juice-shop .'
                }
            }
        }

        stage('Deploy to Server') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    sshagent(credentials: ['app-server']) {
                        sh '''
                            scp -o StrictHostKeyChecking=no temp_repo/webgoat-server/target/webgoat-server-v8.2.0-SNAPSHOT.jar ubuntu@:/WebGoat
                            ssh -o StrictHostKeyChecking=no ubuntu@3.109.152.116 "nohup java -jar /WebGoat/webgoat-server-v8.2.0-SNAPSHOT.jar > /dev/null 2>&1 &"
                        '''
                    }
                }
            }
        }

        stage('Run ZAP DAST Scan (Baseline)') {
            steps {
                echo 'Running ZAP Baseline DAST Scan...'
                sh '''
                    docker run --rm \
                      -v $WORKSPACE:/zap/wrk/:rw \
                      zaproxy/zap-stable \
                      zap-baseline.py -t $TARGET_URL \
                      -r $ZAP_REPORT_HTML -x $ZAP_REPORT_XML -J $ZAP_REPORT_JSON || true
                '''
                archiveArtifacts artifacts: "${ZAP_REPORT_HTML}, ${ZAP_REPORT_XML}, ${ZAP_REPORT_JSON}", onlyIfSuccessful: false
            }
        }
    }

    post {
        always {
            echo 'Cleaning up temporary files...'
            sh '''
                rm -rf temp_repo dependency-check-report trufflehog_report.txt \
                       $ZAP_REPORT_HTML $ZAP_REPORT_XML $ZAP_REPORT_JSON || true
            '''
        }
    }
}
