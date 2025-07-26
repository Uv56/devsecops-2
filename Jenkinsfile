pipeline {
    agent any

    environment {
        ZAP_REPORT_HTML = 'zap_report.html'
        ZAP_REPORT_XML = 'zap_report.xml'
        ZAP_REPORT_JSON = 'zap_report.json'
        TARGET_URL = 'http://98.81.237.97:8080/WebGoat'
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

       /* stage('Secret Scan (TruffleHog)') {
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
                echo 'Running OWASP Dependency-Check using Docker...'
                sh '''
                    mkdir -p dependency-check-report
                    docker run --rm \
                        -v $(pwd)/temp_repo:/src \
                        -v $(pwd)/dependency-check-report:/report \
                        -v $(pwd)/odc-data:/usr/share/dependency-check/data \
                        owasp/dependency-check \
                        --project "Universal-SCA-Scan" \
                        --scan /src \
                        --format "ALL" \
                        --out /report || true
                '''
                archiveArtifacts artifacts: 'dependency-check-report/*', onlyIfSuccessful: false
            }
        }
*/
        stage('SonarQube Scan') {
            steps {
                echo 'Starting SonarQube SAST Scan...'
                withSonarQubeEnv('sonar') {
                    script {
                        def scannerHome = tool name: 'sonar-scanner'
                        sh """
                            cd temp_repo
                            ${scannerHome}/bin/sonar-scanner \\
                              -Dsonar.projectKey=juice-shop \\
                              -Dsonar.sources=. \\
                              -Dsonar.host.url=http://192.168.18.137:9000
                        """
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

        stage('Deploy to App Server') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    sshagent(credentials: ['app-server']) {
                        sh '''
                            echo "Deploying JAR to App Server..."
                            scp -o StrictHostKeyChecking=no temp_repo/webgoat-server/target/webgoat-server-v8.2.0-SNAPSHOT.jar ubuntu@98.81.237.97:/WebGoat
                            ssh -o StrictHostKeyChecking=no ubuntu@98.81.237.97 "pkill -f webgoat || true; nohup java -jar /WebGoat/webgoat-server-v8.2.0-SNAPSHOT.jar > /dev/null 2>&1 &"
                        '''
                    }
                }
            }
        }

        stage('Run ZAP DAST Scan') {
            steps {
                echo 'Running ZAP Full DAST Scan on deployed application...'
                sh '''
                    docker run --rm \
                      -v $WORKSPACE:/zap/wrk/:rw \
                      owasp/zap2docker-stable \
                      zap-full-scan.py -t $TARGET_URL \
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
                rm -rf temp_repo dependency-check-report odc-data trufflehog_report.txt \
                       $ZAP_REPORT_HTML $ZAP_REPORT_XML $ZAP_REPORT_JSON || true
            '''
        }
    }
}
