pipeline {
    agent any 

    parameters {
        string(name: 'THREADS', defaultValue: '10', description: 'Number of total users')
        string(name: 'RAMPUP', defaultValue: '300', description: 'Ramp-up time in seconds')
    }

    environment {
        // Change this to your actual email
        EMAIL_RECIPIENT = "poulomidas89@gmail.com" 
    }

    stages {
        stage('Cleanup') {
            steps {
                sh 'rm -rf results reports' 
                sh 'mkdir -p results reports'
            }
        }

        stage('Run JMeter Test') {
            steps {
                script {
                    // 1. Force permissions so Docker can read the files
                    sh "chmod -R 777 ." 
                    
                    // 2. Use $(pwd) for the volume mount - it is more reliable in Docker
                    // 3. Added --user root to ensure the container has access
                    sh """
                    docker run --rm --user root \
                        -v "\$(pwd):/tests" \
                        -w /tests \
                        justb4/jmeter \
                        -n -t PerformanceTest.jmx \
                        -l results/output.jtl \
                        -e -o reports/ \
                        -Jusers=${params.THREADS} \
                        -Jrampup=${params.RAMPUP}
                    """
                }
            }
        }
        stage('Publish Report') {
            steps {
                // This makes the report viewable directly in Jenkins
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'reports',
                    reportFiles: 'index.html',
                    reportName: 'JMeter Performance Dashboard'
                ])
            }
        }
    }

    post {
        always {
            script {
                // Zip the report to send via email
                zip zipFile: 'jmeter-report.zip', dir: 'reports'
                
                emailext (
                    to: "${env.EMAIL_RECIPIENT}",
                    subject: "Performance Test Report - Build #${env.BUILD_NUMBER}",
                    body: """<h3>Test Execution Completed</h3>
                             <p>Status: ${currentBuild.result ?: 'SUCCESS'}</p>
                             <p>Check the attached zip for the 90th percentile and throughput metrics.</p>
                             <p>Jenkins Build URL: ${env.BUILD_URL}</p>""",
                    attachmentsPattern: 'jmeter-report.zip',
                    mimeType: 'text/html'
                )
            }
            archiveArtifacts artifacts: 'results/*.jtl', allowEmptyArchive: true
        }
    }
}
