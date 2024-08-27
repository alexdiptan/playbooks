def runPlaybook(String playbook) {
    sh """echo '${playbook}' > 1.log"""
}

projectDescription = """<p>Перенос из excel. Скрипт scripts/client_file_import/import.py/p>
        <p><a href='https://huntflow.atlassian.net/wiki/spaces/SUPPORTENGINEERS/pages/2802057217?atlOrigin=eyJpIjoiYzQ5OTNjYzIyNDZmNGVmN2E3YzQzMDllNTNhZTJlMTMiLCJwIjoiY29uZmx1ZW5jZS1jaGF0cy1pbnQifQ<width='1'>Инструкция для получения url и пароля из owncloud</p>
        <p><a href='https://huntflow.atlassian.net/browse/DEVOPS-926<width='1'>Jira Ticket</p>"""

// Build info
currentBuild.displayName = "Перенос данных на ${params.INVENTORY_FILE} организации ${params.ACCOUNT_ID} из excel #${currentBuild.number}"
currentBuild.description = "Импорт из эксель"

pipeline {
    agent any
    
    parameters {
        string(name: 'USER_ID', defaultValue: '5698', description: 'ID пользователя от которого будет запущен импорт')
        validatingString(name: 'OWNCLOUD_URL', defaultValue: 'https://securedata.huntflow.ru/s/X5Mwqu0hjZ10N17', description: 'Ссылка на excel-файл', regex: '^https://securedata.huntflow.ru/s/.*$') 
        
    }
    
    environment {
        STEP = 'FOO'
        CURRENT_DATE = new Date().format("ddMMyyyy_HHmmssS")
    }
    
    options {
        buildDiscarder(logRotator(numToKeepStr: '5'))

    }
    
    stages {
        stage("⬇️ Checkout repositories") {
            steps{
                retry(1) {
                    checkout([
                        $class: 'GitSCM',
                        userRemoteConfigs: [[
                            url: 'https://github.com/alexdiptan/playbooks.git',
                            credentialsId: 'jenkins_git_token'
                        ]],
                        branches: [[name: 'master']],
                        doGenerateSubmoduleConfigurations: false,
                        extensions: [
                            [$class: 'CloneOption', depth: 1, noTags: false, reference: '/opt/gitcache/playbooks.git', shallow: true, timeout: 20]
                        ],
                        submoduleCfg: []
                    ])
                }
            }
        }
        stage('👮 Add huntflow_import to org') {
            steps {
                script {
                    if (params.USER_ID == 'huntflow_import_user_id') {
                        echo 'Use default user'
                        runPlaybook('user_control')
                    } else {
                        echo 'Skip this step'
                    }
                }
            }
        }        
        stage('Check') {
            steps {
                script {
                    println CURRENT_DATE
                    def all_stages = [
                        '🔧 Consistency check before': 'consistency_check',
                        '📲 Download file': 'download_file',
                        '📥 Import': 'files_import',
                        '👤 Applicants reindex': 'applicants_reindex',
                        '📜 Vacancies reindex': 'vacancies_reindex',
                        '👯‍ Find duplicates': 'find_duplicates',
                        '🔧 Consistency check after': 'consistency_check'
                    ]
                    all_stages.each { stageName, playbook ->
                        stage(stageName) {
                            runPlaybook(playbook)
                        }
                    }
                }
            }
        }
    }
    post {
            always {
                archiveArtifacts "**/*"
            }
            success {
                println 'Success'
            }
            failure {
                sh 'echo 321'
            }
    }
}
