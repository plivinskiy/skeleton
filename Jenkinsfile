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
            name: 'reference_type',
            choices: [
                'branch',
                'tag'
            ]
        )
        string(
            name: 'reference',
            defaultValue: 'v1.3.0-5.8.0'
        )
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
                    if ( "${env.reference_type}" == 'branch' ) {
                        env.build_from = "refs/heads/${env.reference}"
                    }

                    else if ( "${env.reference_type}" == 'tag' ) {
                        env.build_from = "refs/tags/${env.reference}"
                    }
                }

                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "${env.build_from}"]],
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
                                    execCommand: 'mkdir test && tar -zxvf build/build/output.tar.gz -C ./test && rm -rf build/build/output.tar.gz',
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
