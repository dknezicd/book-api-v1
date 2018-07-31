pipeline {
    agent any 
    tools {
        maven "maven-3.x"
        nodejs "nodejs-8.x"
    }
    stages {
        stage('Deploy And Test') {
            steps {
                script {
                    // feature branches are deployed to environment used for development
                    if(env.BRANCH_NAME.startsWith('feature')) {
                        profile = 'feature'
                    } else  {
                        profile = env.BRANCH_NAME    
                    }
                }
                withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'apigee-credentials',
                            usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {     
                     configFileProvider([configFile(fileId: 'apigee-settings', variable: 'APIGEE_SETTINGS')]) {
                        sh "mvn install -s${APIGEE_SETTINGS} -P${profile} -Ddescription.suffix=\" branch: ${BRANCH_NAME} commit: ${GIT_COMMIT}\" -Dusername=${USERNAME} -Dpassword=${PASSWORD}"
                    }
 
                }
            }
        }
        stage('Update smartdocs') {
            withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'devportal-credentials',
                            usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {     
                withCredentials([[$class: 'StringBinding', credentialsId: 'devportal-cronkey',
                            usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {     
                        sh "./update_smartdocs.sh ${USERNAME} ${PASSWORD} http://dev-ylesyuk.devportal.apigee.io/ ${CRON_KEY}"
                }
            }
        }
    }
    post {
        always {
            junit 'test/unit/test-report.xml'
            cucumber 'test/integration/report.json'
        }
    }
}
