pipeline {
    agent none
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'python:2-alpine'
                }
            }
            steps {
                sh 'python -m py_compile sources/add2vals.py sources/calc.py'
                stash(name: 'compiled-results', includes: 'sources/*.py*')
            }

        }

    stage('Wait for 30 seconds') {
        steps {
            sleep time: 30, unit: 'SECONDS'
        }
    }

        stage('Test') {
            agent {
                docker {
                    image 'qnib/pytest'
                }
            }
            steps {
                sh 'py.test --junit-xml test-reports/results.xml sources/test_calc.py'
            }
            post {
                always {
                    junit 'test-reports/results.xml'
                }
            }
        }

        stage('Manual Approval') {
            steps {
                input message: 'Proceed with delivery?', ok: 'Deploy'
            }
        }

    stage('Deploy') {
    agent any
    environment {
        VOLUME = '$(pwd)/sources:/src'
        IMAGE = 'cdrx/pyinstaller-linux:python2'
    }
    steps {
        script {
            // Menjalankan aplikasi dengan Docker Compose
            sh "docker-compose -f docker-compose.yml up -d"
            // Menunggu 1 menit (60 detik)
            sleep 60
            // Menghentikan kontainer dengan Docker Compose
            sh "docker-compose -f docker-compose.yml down"
        }
    }
    post {
        success {
            // Arsipkan hasil build
            archiveArtifacts "${env.BUILD_ID}/sources/dist/add2vals"
            // Bersihkan build artifacts
            sh "docker run --rm -v ${VOLUME} ${IMAGE} 'rm -rf build dist'"
        }
    }
}


    }
}