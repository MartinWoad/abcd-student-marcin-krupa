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
                    docker cp zap:/zap/wrk/results/zap_html_report.html /tmp/zap_html_report.html
                    docker cp /tmp/zap_html_report.html abcd-lab:/"${WORKSPACE}"/results/zap_html_report.html
                    docker cp zap:/zap/wrk/results/zap_xml_report.xml /tmp/zap_xml_report.xml
                    docker cp /tmp/zap_xml_report.xml abcd-lab:/"${WORKSPACE}"/results/zap_xml_report.xml
                '''
            }
            post
            {
                always
                {
                    sh '''
                        rm /tmp/zap_html_report.html
                        rm /tmp/zap_xml_report.xml
                        docker stop zap juice-shop
                        docker rm zap juice-shop
                    '''
                }
            }
        }
    }
}
