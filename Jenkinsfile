pipeline {
    agent any

    environment {
        DEPENDENCY_CHECK = '/opt/dependency-check/dependency-check/bin/dependency-check.sh'
        NIKTO_REPORT = 'nikto_report.html'
        TARGET_URL = 'http://192.168.18.137:3000'
    }

    stages {

        /*
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
                    cd temp_repo
                    trufflehog --regex --entropy=True --max_depth=10 . > ../trufflehog_report.txt || true
                '''
                archiveArtifacts artifacts: 'trufflehog_report.txt', onlyIfSuccessful: false
            }
        }

        stage('Dependency Check (OWASP)') {
            steps {
                echo 'Running OWASP Dependency-Check...'
                sh '''
                    mkdir -p dependency-check-report
                    cd temp_repo
                    $DEPENDENCY_CHECK --project "Universal-SCA-Scan" --scan . --format ALL --out ../dependency-check-report || true
                    cd ..
                '''
                archiveArtifacts artifacts: 'dependency-check-report/*', onlyIfSuccessful: false
            }
        }

        stage('SonarQube Scan') {
            steps {
                echo 'Starting SonarQube SAST Scan...'
                withSonarQubeEnv('sonar') {
                    script {
                        def scannerHome = tool name: 'sonar-scanner'
                        sh """
                            cd temp_repo
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=juice-shop \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=http://192.168.81.128:9000
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

        stage('Deploy to Server') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    sshagent(credentials: ['app-server']) {
                        sh '''
                            scp -o StrictHostKeyChecking=no temp_repo/webgoat-server/target/webgoat-server-v8.2.0-SNAPSHOT.jar ubuntu@3.109.152.116:/WebGoat
                            ssh -o StrictHostKeyChecking=no ubuntu@3.109.152.116 "nohup java -jar /WebGoat/webgoat-server-v8.2.0-SNAPSHOT.jar > /dev/null 2>&1 &"
                        '''
                    }
                }
            }
        }
        */

        stage('Run Nikto DAST Scan') {
            steps {
                echo 'Running Nikto DAST Scan...'
                sh '''
                    rm -rf nikto
                    git clone https://github.com/sullo/nikto.git
                    cd nikto/program
                    chmod +x nikto.pl
                    ./nikto.pl -h $TARGET_URL -o $WORKSPACE/$NIKTO_REPORT -Format html || true
                '''
                archiveArtifacts artifacts: "${NIKTO_REPORT}", onlyIfSuccessful: false
            }
        }
    }

    post {
        always {
            echo 'Cleaning up temporary files...'
            sh '''
                rm -rf temp_repo dependency-check-report trufflehog_report.txt nikto_report.html nikto || true
            '''
        }
    }
}
