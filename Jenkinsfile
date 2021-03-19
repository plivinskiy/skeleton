pipeline {
    agent {
        node {
            label 'master'
        }
    }

    options {
        copyArtifactPermission('MUCK-pimcore-test')
    }

    parameters {
        choice(
            name: 'JK_GIT_REFERENCE_TEST',
            description: '# Git Branch or Tag to checkout',
            choices: [
                'branch',
                'tag'
            ]
        )

        credentials(name: 'JK_GIT_CREDENTIAL', description: '# GIT Repository Credentials', defaultValue: 'ec95ffe3-1911-452d-8366-65f22a18a6ca', credentialType: "Any", required: true )

        gitParameter branchFilter: 'origin/(.*)', defaultValue: 'master', name: 'JK_GIT_REFERENCE', type: 'PT_BRANCH'
    }

    stages {
        stage('Build') {
            agent {
                dockerfile {
                    filename 'Dockerfile'
                    dir 'docker/'
                    label 'master'
                    args '-v /root/jenkins_home/.composer:/.composer'
                }
            }

            steps {

                script {
                    if ( "${env.JK_GIT_REFERENCE}" == 'branch' ) {
                        env.build_from = "refs/heads/${env.reference}"
                    }

                    else if ( "${env.JK_GIT_REFERENCE}" == 'tag' ) {
                        env.build_from = "refs/tags/${env.reference}"
                    }
                }

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "refs/heads/${env.JK_GIT_REFERENCE}"]],
                    userRemoteConfigs: [[name: 'source', credentialsId:"559c13f8-1f51-4e26-ab45-5a71eba00ef3", url: "https://github.com/plivinskiy/skeleton.git"]]
                ])

                sh 'composer install --optimize-autoloader --no-interaction'
                sh 'php bin/console cache:clear'
                sh 'rm -rf .git/'
                sh 'mkdir build'
                sh 'tar --exclude=build/output.tar.gz -zcf build/output.tar.gz .'

                archiveArtifacts artifacts: 'build/output.tar.gz', followSymlinks: false, onlyIfSuccessful: true
            }
        }
        stage('Deploy') {
            steps {
               sh "pwd"

               copyArtifacts filter: 'build/output.tar.gz', fingerprintArtifacts: true, projectName: 'pimcore-skeleton', selector: lastSuccessful()

               sh "ls -la"

               sshPublisher(
                publishers:
                    [
                        sshPublisherDesc(
                            configName: 'Tagwork-K1',
                            transfers: [
                                sshTransfer(
                                    cleanRemote: false,
                                    excludes: '',
                                    execCommand: 'rm -rf test && mkdir test && tar -zxvf build/build/output.tar.gz -C ./test && rm -rf build/build/output.tar.gz',
                                    execTimeout: 120000,
                                    flatten: false,
                                    makeEmptyDirs: false,
                                    noDefaultExcludes: false,
                                    patternSeparator: '[, ]+',
                                    remoteDirectory: 'build',
                                    remoteDirectorySDF: false,
                                    removePrefix: '',
                                    sourceFiles: 'build/output.tar.gz'
                                )
                            ],
                            usePromotionTimestamp: false,
                            useWorkspaceInPromotion: false,
                            verbose: true
                        )
                    ]
                )
            }
        }
    }
}
