pipeline {
    agent {
        dockerfile {
            filename 'Dockerfile'
            dir 'docker/'
            label 'master'
            args '-v /root/jenkins_home/.composer:/.composer'
        }
    }

    options {
        copyArtifactPermission("${env.JOB_NAME}")
    }

    parameters {

        choice(name: 'JK_SSH_HOST', choices: ['62.146.146.113'], description: 'Deploy Destination: Host')

        choice(name: 'JK_REMOTE_DESTINATION', choices: ['/var/www/pimcore'], description: 'Deploy Destination: Directory')

        choice(name: 'JK_SSH_USER', choices: ['t1admin'], description: 'Deploy Destination: User')

        choice(name: 'JK_ENV_NAME', choices: ['staging'], description: 'Environments name')

        credentials(name: 'JK_SSH_CREDENTIAL', description: 'SSH Jenkins Private Key', defaultValue: 'JenkinsPrivateKey', credentialType: "Any", required: true )

        credentials(name: 'JK_GIT_CREDENTIAL', description: 'GIT Repository Credentials', defaultValue: 'gitea-tagwork', credentialType: "Any", required: true )

        gitParameter branchFilter: 'origin/(.*)', defaultValue: 'master', description: 'GIT Branch to build the project', name: 'JK_GIT_REFERENCE', type: 'PT_BRANCH'
    }

    stages {

        stage('Building') {

            steps {

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "refs/heads/${env.JK_GIT_REFERENCE}"]],
                    userRemoteConfigs: [[name: 'source', credentialsId:"${env.JK_GIT_CREDENTIAL}", url: "${env.GIT_URL}"]]
                ])

//                 sh 'composer install --optimize-autoloader --no-interaction'
//                 sh 'php bin/console cache:clear'
                sh 'rm -rf .git/'
//                 sh 'rm -rf var/cache'
                sh "if [ -d 'build' ]; then rm -rf build; fi"
                sh 'mkdir build'
                sh "tar --exclude=build/artifact-${env.BUILD_NUMBER}.tar.gz -zcf build/artifact-${env.BUILD_NUMBER}.tar.gz ."

                archiveArtifacts artifacts: "build/artifact-${env.BUILD_NUMBER}.tar.gz", followSymlinks: false, onlyIfSuccessful: true
            }
        }

        stage('Testing') {
            steps {
                sh "echo 'run tests'"
            }
        }

        stage('Deploying') {
            steps {

                script {
                    def remote = [:]
                    remote.name = "Deploy Destination"
                    remote.host = "${env.JS_SSH_HOST}"
                    remote.user = "${env.JS_SSH_USER}"
                    remote.allowAnyHosts = true

                    sh "test credentials: ${env.JK_SSH_CREDENTIAL}"

                    sh "host: ${env.JS_SSH_HOST}"

                    sh "user: ${env.JS_SSH_USER}"

                    withCredentials([sshUserPrivateKey(credentialsId: "JenkinsPrivateKey", keyFileVariable: 'identity', passphraseVariable: 'passphrase', usernameVariable: 'userName')]) {
                        remote.identityFile = identity
                        remote.passphrase = passphrase

                        stage("Copy to remote Artifacts Storage") {
                            sshCommand remote: remote, command: "if [ ! -d '${env.JK_REMOTE_DESTINATION}' ]; then mkdir ${env.JK_REMOTE_DESTINATION}/artifacts; fi"
                            sshPut remote: remote, from: 'build/artifact.tar.gz', into: "${env.JK_REMOTE_DESTINATION}/artifacts/artifact-${env.BUILD_NUMBER}.tar.gz"
                            sshCommand remote: remote, command: "ls -la ${env.JK_REMOTE_DESTINATION}/artifacts"
                            sshCommand remote: remote, command: "if [ ! -d '${env.JK_REMOTE_DESTINATION}/release' ]; then mkdir ${env.JK_REMOTE_DESTINATION}/release; fi"
                            sshCommand remote: remote, command: "if [ ! -d '${env.JK_REMOTE_DESTINATION}/release/${env.BUILD_NUMBER}' ]; then mkdir ${env.JK_REMOTE_DESTINATION}/release/${env.BUILD_NUMBER}; fi"
                            sshCommand remote: remote, command: "tar -zxf ${env.JK_REMOTE_DESTINATION}/artifacts/artifact-${env.BUILD_NUMBER}.tar.gz --directory ${env.JK_REMOTE_DESTINATION}/release/${env.BUILD_NUMBER}"
                            sshCommand remote: remote, command: "find ${env.JK_REMOTE_DESTINATION}/artifacts -maxdepth 1 -type f | xargs -x ls -t | awk 'NR>5' | xargs -L1 rm -rf"
                            sshCommand remote: remote, command: "cp ${env.JK_REMOTE_DESTINATION}/release/${env.BUILD_NUMBER}/var/config/system_${env.JK_ENV_NAME}.php ${env.JK_REMOTE_DESTINATION}/release/${env.BUILD_NUMBER}/var/config/system.php "
                        }
                    }

                }

            }
        }

        stage('Switching'){
            steps {

                script {
                    def remote = [:]
                    remote.name = "Deploy Destination"
                    remote.host = "${env.JK_SSH_HOST}"
                    remote.user = "${env.JK_SSH_USER}"
                    remote.allowAnyHosts = true

                    withCredentials([sshUserPrivateKey(credentialsId: "${env.JK_SSH_CREDENTIAL}", keyFileVariable: 'identity', passphraseVariable: 'passphrase', usernameVariable: 'userName')]) {
                        remote.identityFile = identity
                        remote.passphrase = passphrase

                        stage("Prepare symlinks") {
                            // web/var
                            sshCommand remote: remote, command: "rm -rf ${env.JK_REMOTE_DESTINATION}release/${env.BUILD_NUMBER}/web/var"
                            sshCommand remote: remote, command: "ln -sf ${env.JK_REMOTE_DESTINATION}release/${env.BUILD_NUMBER}/web/var ${env.JK_REMOTE_DESTINATION}/shared/web/var"

                            // web/assets
                            sshCommand remote: remote, command: "rm -rf ${env.JK_REMOTE_DESTINATION}release/${env.BUILD_NUMBER}/web/assets"
                            sshCommand remote: remote, command: "ln -sf ${env.JK_REMOTE_DESTINATION}release/${env.BUILD_NUMBER}/web/assets ${env.JK_REMOTE_DESTINATION}/shared/web/assets"

                            // var/logs
                            sshCommand remote: remote, command: "rm -rf ${env.JK_REMOTE_DESTINATION}release/${env.BUILD_NUMBER}/var/logs"
                            sshCommand remote: remote, command: "ln -sf ${env.JK_REMOTE_DESTINATION}release/${env.BUILD_NUMBER}/var/logs ${env.JK_REMOTE_DESTINATION}/shared/var/logs"

                            // var/sessions
                            sshCommand remote: remote, command: "rm -rf ${env.JK_REMOTE_DESTINATION}release/${env.BUILD_NUMBER}/var/sessions"
                            sshCommand remote: remote, command: "ln -sf ${env.JK_REMOTE_DESTINATION}release/${env.BUILD_NUMBER}/var/sessions ${env.JK_REMOTE_DESTINATION}/shared/var/sessions"

                            // var/tmp
                            sshCommand remote: remote, command: "rm -rf ${env.JK_REMOTE_DESTINATION}release/${env.BUILD_NUMBER}/var/tmp"
                            sshCommand remote: remote, command: "ln -sf ${env.JK_REMOTE_DESTINATION}release/${env.BUILD_NUMBER}/var/tmp ${env.JK_REMOTE_DESTINATION}/shared/var/tmp"

                            // var/versions
                            sshCommand remote: remote, command: "rm -rf ${env.JK_REMOTE_DESTINATION}release/${env.BUILD_NUMBER}/var/versions"
                            sshCommand remote: remote, command: "ln -sf ${env.JK_REMOTE_DESTINATION}release/${env.BUILD_NUMBER}/var/versions ${env.JK_REMOTE_DESTINATION}/shared/var/versions"

                            sshCommand remote: remote, command: "ln -sf ${env.JK_REMOTE_DESTINATION}htdocs ${env.JK_REMOTE_DESTINATION}/release/${env.BUILD_NUMBER}"
                        }
                    }

                }


            }
        }

    }
}