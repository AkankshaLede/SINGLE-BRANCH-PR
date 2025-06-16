pipeline {
    agent any
    environment {
        // Generate a timestamp for tagging
        TIMESTAMP = "${new Date().format('yyyyMMdd-HHmm')}"
        TAG_NAME = "cfg-change-${TIMESTAMP}" // Define TAG_NAME in the environment
    }

    stages {
        stage('Check for CFG Changes') {
            steps {
                script {
                    def changedFiles = sh(script: 'git diff --name-only HEAD~1', returnStdout: true).trim()
                    if (changedFiles) {
                        def cfgFilesChanged = changedFiles.split('\n').any { it.endsWith('.cfg') }
                        if (cfgFilesChanged) {
                            echo "Changes detected in .cfg files."
                            env.CFG_CHANGES_DETECTED = true
                        } else {
                            echo "No changes detected in .cfg files."
                            env.CFG_CHANGES_DETECTED = false
                        }
                    } else {
                        echo "No changes found in the last commit."
                        env.CFG_CHANGES_DETECTED = false
                    }
                }
            }
        }

        stage('Create Pull Request (if CFG changed)') {
            when {
                environment name: 'CFG_CHANGES_DETECTED', value: 'true'
            }
            steps {
                withCredentials([string(credentialsId: 'github-pat-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        unset GITHUB_TOKEN
                        echo "$GITHUB_TOKEN" | gh auth login --with-token
                        gh pr create --title "Automated PR: Changes in CFG files" --body "This PR was automatically created due to changes detected in .cfg files." --base main --head ${GIT_BRANCH}
                    '''
                }
            }
        }

        stage('Create GitHub Release (if CFG changed)') {
            when {
                environment name: 'CFG_CHANGES_DETECTED', value: 'true'
            }
            steps {
                withCredentials([string(credentialsId: 'github-pat-token', variable: 'GITHUB_TOKEN')]) {
                    sh '''
                        unset GITHUB_TOKEN
                        echo "$GITHUB_TOKEN" | gh auth login --with-token
                        gh release create "$TAG_NAME" \
                            --title "CFG Changes Release - $TAG_NAME" \
                            --notes "Automated release created due to .cfg changes."
                    '''
                }
            }
        }
    }
}


// pipeline {
//     agent any
//     environment {
//         // Generate a timestamp for tagging
//         TIMESTAMP = "${new Date().format('yyyyMMdd-HHmm')}"
//         TAG_NAME = "cfg-change-${TIMESTAMP}" // Define TAG_NAME in the environment
//     }
//
//     stages {
//         stage('Check for CFG Changes') {
//             steps {
//                 script {
//                     def changedFiles = sh(script: 'git diff --name-only HEAD~1', returnStdout: true).trim()
//                     if (changedFiles) {
//                         def cfgFilesChanged = changedFiles.split('\n').any { it.endsWith('.cfg') }
//                         if (cfgFilesChanged) {
//                             echo "Changes detected in .cfg files."
//                             env.CFG_CHANGES_DETECTED = "true"
//                         } else {
//                             echo "No changes detected in .cfg files."
//                             env.CFG_CHANGES_DETECTED = "false"
//                         }
//                     } else {
//                         echo "No changes found in the last commit."
//                         env.CFG_CHANGES_DETECTED = "false"
//                     }
//                 }
//             }
//         }
//
//         stage('Create Pull Request (if CFG changed)') {
//             when {
//                 environment name: 'CFG_CHANGES_DETECTED', value: 'true'
//             }
//             steps {
//                 withCredentials([string(credentialsId: 'github-pat-token', variable: 'GITHUB_TOKEN')]) {
//                     sh '''
//                         unset GITHUB_TOKEN
//                         echo "$GITHUB_TOKEN" | gh auth login --with-token
//                         gh pr create --title "Automated PR: Changes in CFG files" \
//                                      --body "This PR was automatically created due to changes detected in .cfg files." \
//                                      --base main --head ${GIT_BRANCH} || echo "PR might already exist. Skipping creation."
//                     '''
//                 }
//             }
//         }
//
//         stage('Create GitHub Release (if CFG changed)') {
//             when {
//                 environment name: 'CFG_CHANGES_DETECTED', value: 'true'
//             }
//             steps {
//                 withCredentials([string(credentialsId: 'github-pat-token', variable: 'GITHUB_TOKEN')]) {
//                     sh '''
//                         unset GITHUB_TOKEN
//                         echo "$GITHUB_TOKEN" | gh auth login --with-token
//                         gh release create "$TAG_NAME" \
//                             --title "CFG Changes Release - $TAG_NAME" \
//                             --notes "Automated release created due to .cfg changes." || echo "Release might already exist. Skipping creation."
//                     '''
//                 }
//             }
//         }
//     }
// }
