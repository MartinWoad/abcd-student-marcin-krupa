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
            }
            post
            {
                always
                {
                    sh '''
                        docker exec zap scp /zap/wrk/results/zap_html_report.html abcd-lab:/"${WORKSPACE}"/results/zap_html_report.html
                        docker exec zap scp /zap/wrk/results/zap_xml_report.xml abcd-lab:/"${WORKSPACE}"/results/zap_xml_report.xml
                    '''
                }
                always
                {
                    sh '''
                        docker stop zap juice-shop
                    '''
                }
            }
        }
    }
}
