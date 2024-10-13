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
                    git credentialsId: 'github-pat', url: 'https://github.com/MartinWoad/abcd-student-marcin-krupa', branch: 'main'
                }
            }
        }
        stage('[ZAP] Baseline passive-scan')
        {
            steps
            {
                script
                {
                    sh
                    '''
                        docker run --name juice-shop -d \
                        -p 3000:3000 \
                        bkimminich/juice-shop
                        sleep 5
                    '''
                    sh
                    '''
                        docker run --name zap \
                        --add-host=host.docker.internal:host-gateway \
                        -v /path/to/dir/with/passive/scan/yaml:/zap/wrk/:rw
                        -t ghcr.io/zaproxy/zaproxy:stable bash -c \
                        "zap.sh -cmd -addonupdate; \
                        zap.sh -cmd \
                        -addoninstall communityScripts \
                        -addoninstall pscanrulesAlpha \
                        -addoninstall pscanrulesBeta \
                        -autorun /zap/wrk/passive_scan.yaml" \
                        || true
                    '''
                }
            }
            post
            {
                always
                {
                    sh
                    '''
                        docker cp zap:/zap/wrk/zap_html_report.html ${WORKSPACE}/results/zap_html_report.html
                        docker cp zap:/zap/wrk/zap_xml_report.xml ${WORKSPACE}/results/zap_xml_report.xml
                        docker stop zap juice-shop
                    '''
                }
            }
        }
    }
}
