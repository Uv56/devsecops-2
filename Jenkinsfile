pipeline {
    agent any

    environment {
        DEPENDENCY_CHECK = '/opt/dependency-check/dependency-check/bin/dependency-check.sh'
        ZAP_REPORT_HTML  = 'zap_report.html'
        ZAP_REPORT_XML   = 'zap_report.xml'
        ZAP_REPORT_JSON  = 'zap_report.json'
        TARGET_URL       = 'http://192.168.18.137:3000' // Replace with actual target
        DEFECTDOJO_URL   = 'http:192.168.18.137//:8081'      // Replace with your local DefectDojo URL
        ENGAGEMENT_ID    = '111'                          // Replace with your actual engagement ID
    }

    stages {
        stage('Clone Repository') {
            steps {
                echo 'Cloning full Git history...'
                sh '''
                    rm -rf temp_repo
                    git clone --depth=1000 https://github.com/Akashsonawane571/devsecops-test.git temp_repo
                '''
            }
        }

        
        stage('Secret Scan (TruffleHog)') {
            steps {
                echo 'Running TruffleHog on latest commit only...'
                sh '''
                    cd temp_repo
                    trufflehog --regex --entropy=True --max_depth=10 . > ../trufflehog_report.json || true
                    cd ..
                '''
                archiveArtifacts artifacts: 'trufflehog_report.json', onlyIfSuccessful: false
            }
        }

        stage('Dependency Check (OWASP)') {
            steps {
                echo 'Running OWASP Dependency-Check...'
                sh '''
                    mkdir -p dependency-check-report
                    /opt/dependency-check/dependency-check/bin/dependency-check.sh \
                      --project "Universal-SCA-Scan" \
                      --scan temp_repo \
                      --format ALL \
                      --enableExperimental \
                      --out dependency-check-report || true
                '''
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

        
        stage('Run Built Docker Image Locally') {
            steps {
                echo 'Running built Docker image locally...'
                sh '''
                    docker rm -f juice-shop-running || true
                    docker run -d --name juice-shop-running -p 3000:3000 juice-shop
                    echo "Waiting for container to be ready..."
                    sleep 15
                '''
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
        

        // ✅ NEW STAGE: Upload to DefectDojo
        stage('Upload TruffleHog Report to DefectDojo') {
            steps {
                withCredentials([string(credentialsId: 'DEFECTDOJO_API_TOKEN', variable: 'DD_API_KEY')]) {
                    sh '''
                        if [ -f trufflehog_report.json ]; then
                            curl -X POST "http://192.168.18.137:8081/api/v2/import-scan/" \
                              -H "Authorization: Token $DD_API_KEY" \
                              -F "file=@trufflehog_report.json" \
                              -F "scan_type=Trufflehog Scan" \
                              -F "engagement=$ENGAGEMENT_ID" \
                              -F "active=true" -F "verified=true" -F "close_old_findings=true"
                        fi
                    '''
                }
            }
        }
        
        stage('Upload Dependency-Check Report to DefectDojo') {
            steps {
                withCredentials([string(credentialsId: 'DEFECTDOJO_API_TOKEN', variable: 'DD_API_KEY')]) {
                    sh '''
                        if [ -f dependency-check-report/dependency-check-report.xml ]; then
                            curl -X POST "http://192.168.18.137:8081/api/v2/import-scan/" \
                              -H "Authorization: Token $DD_API_KEY" \
                              -F "file=@dependency-check-report/dependency-check-report.xml" \
                              -F "scan_type=Dependency Check Scan" \
                              -F "engagement=$ENGAGEMENT_ID" \
                              -F "active=true" -F "verified=true" -F "close_old_findings=true"
                        fi
                    '''
                }
            }
        }
        
        stage('Upload ZAP Report to DefectDojo') {
            steps {
                withCredentials([string(credentialsId: 'DEFECTDOJO_API_TOKEN', variable: 'DD_API_KEY')]) {
                    sh '''
                        if [ -f $ZAP_REPORT_XML ]; then
                            curl -X POST "http://192.168.18.137:8081/api/v2/import-scan/" \
                              -H "Authorization: Token $DD_API_KEY" \
                              -F "file=@$ZAP_REPORT_XML" \
                              -F "scan_type=ZAP Scan" \
                              -F "engagement=$ENGAGEMENT_ID" \
                              -F "active=true" -F "verified=true" -F "close_old_findings=true"
                        fi
                    '''
                }
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
