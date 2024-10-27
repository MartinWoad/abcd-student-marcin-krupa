pipeline
{
    agent any
    options
    {
        skipDefaultCheckout(true)
    }
    stages
    {
        stage('Code checkout from GitHub')
        {
            steps
            {
                script
                {
                    cleanWs()
                    sh 'git config --global http.postBuffer 524288000'
                    git credentialsId: 'github-pat', url: 'https://github.com/MartinWoad/abcd-student-marcin-krupa', branch: 'main'
                }
            }
        }
        stage('Prepare')
        {
            steps {
                sh 'docker exec abcd-lab mkdir -p "${WORKSPACE}"/results'
            }
        }
        stage('ZAP Passive Scan')
        {
            steps
            {
                sh '''
                    docker run --name juice-shop -d \
                    -p 3000:3000 \
                    bkimminich/juice-shop
                    sleep 5
                '''
                sh '''
                    docker run --name zap \
                    --add-host=host.docker.internal:host-gateway \
                    -v /home/marcin/ABC-DevSecOps/abcd-student-marcin-krupa/.zap:/zap/wrk/:rw \
                    -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                    "zap.sh -cmd -addonupdate; \
                    zap.sh -cmd \
                    -addoninstall communityScripts \
                    -addoninstall pscanrulesAlpha \
                    -addoninstall pscanrulesBeta \
                    -autorun /zap/wrk/passive.yaml" \
                    || true
                '''
                sh '''
                    docker cp zap:/zap/wrk/reports/zap_html_report.html /tmp/zap_html_report.html
                    docker cp /tmp/zap_html_report.html abcd-lab:/"${WORKSPACE}"/results/zap_html_report.html
                    docker cp zap:/zap/wrk/reports/zap_xml_report.xml /tmp/zap_xml_report.xml
                    docker cp /tmp/zap_xml_report.xml abcd-lab:/"${WORKSPACE}"/results/zap_xml_report.xml
                '''
            }
            post
            {
                always
                {
                    sh '''
                        rm /tmp/zap_html_report.html || true
                        rm /tmp/zap_xml_report.xml || true
                        docker stop zap juice-shop || true
                        docker rm zap juice-shop || true
                    '''
                }
            }
        }
        stage('SCA Scan') {
            steps
            {
                sh 'osv-scanner scan --lockfile package-lock.json --format json --output results/sca-osv-scanner.json || [ $? -eq 1 ]'
            }
        }
        stage('TruffleHog Scan') {
            steps
            {
                sh 'trufflehog git https://github.com/MartinWoad/abcd-student-marcin-krupa --branch main --json --only-verified > results/trufflehog_report.json'
            }
        }
        stage('Archive Artifacts')
        {
                steps
                {
                    archiveArtifacts artifacts: 'results/zap_html_report.html, results/zap_xml_report.xml, results/sca-osv-scanner.json, results/trufflehog_report.json', allowEmptyArchive: true
                }
        }
        /*stage('Publish to DefectDojo')
        {
            steps
            {
                defectDojoPublisher(artifact: 'results/zap_xml_report.xml',
                    productName: 'Juice Shop',
                    scanType: 'ZAP Scan',
                    engagementName: 'marcin.krupa.96@gmail.com')
                defectDojoPublisher(artifact: 'results/sca-osv-scanner.json',
                    productName: 'Juice Shop',
                    scanType: 'OSV Scan',
                    engagementName: 'marcin.krupa.96@gmail.com')
                defectDojoPublisher(artifact: 'results/trufflehog_report.json',
                    productName: 'Juice Shop',
                    scanType: 'Trufflehog Scan',
                    engagementName: 'marcin.krupa.96@gmail.com')
            }
        }*/
    }
}
