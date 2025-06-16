pipeline {
    agent any

    environment {
        BRANCH_NAME = "${env.BRANCH_NAME}"
        IS_PR = false
    }

    stages {
        stage('Determine Event Type') {
            steps {
                script {
                    if (env.CHANGE_BRANCH) {
                        IS_PR = true
                        BRANCH_NAME = env.CHANGE_BRANCH
                        echo "📦 Pull Request from branch: ${BRANCH_NAME}"
                    } else if (env.BRANCH_NAME) {
                        echo "🔁 Branch build: ${env.BRANCH_NAME}"
                    } else {
                        error("Unsupported event: Could not detect branch from environment")
                    }

                    def allowedBranches = ['main', 'branch1','branch2']
                    if (!allowedBranches.contains(BRANCH_NAME)) {
                        echo "⚠️ Skipping branch ${BRANCH_NAME} as it is not in the allowed list."
                        currentBuild.result = 'SUCCESS'
                        error("Branch ${BRANCH_NAME} is not allowed, skipping the build.")
                    }
                }
            }
        }

        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Detect Changed .cfg Files') {
            steps {
                script {
                    withChecks(name: 'CFG Change Detection') {
                        try {
                            echo "📄 Detecting changed .cfg files..."
                            sh "git fetch origin main"

                            def changedCfgFiles = sh(script: "git diff --name-only origin/main...HEAD | grep '\\.cfg\$' || true", returnStdout: true).trim()
                            // def changedCfgFiles = sh(script: 'git diff --name-only origin/main...HEAD | grep \'\.cfg$\' || true', returnStdout: true).trim()

                            if (!changedCfgFiles) {
                                echo "✅ No .cfg files changed. Skipping comparison stage."
                                currentBuild.result = 'SUCCESS'
                                skipRemainingStages = true
                            } else {
                                env.CFG_CHANGED_FILES = changedCfgFiles.replaceAll('\\n', ',')
                                echo "🔍 Changed CFG files: ${env.CFG_CHANGED_FILES}"
                            }
                        } catch (err) {
                            echo "⚠️ Failed during CFG change detection"
                            error("Change detection failed: ${err}")
                        }
                    }
                }
            }
        }

        stage('Run CFG Comparison') {
            when {
                expression { return !env.skipRemainingStages }
            }
            steps {
                script {
                    withChecks(name: 'CFG Value Comparison') {
                        try {
                            sh '''
                                set -e
                                python3 -m venv venv
                                . venv/bin/activate
                                python utils/check_cfg_params.py \
                                    --input_files "${CFG_CHANGED_FILES}" \
                                    --default_file test_data/expected.cfg
                            '''
                        } catch (err) {
                            echo "❌ CFG validation failed"
                            error("CFG value comparison failed: ${err}")
                        }
                    }
                }
            }
        }
    }

    post {
        success {
            echo "✅ Pipeline completed successfully for branch: ${BRANCH_NAME}"
        }
        failure {
            echo "❌ Pipeline failed for branch: ${BRANCH_NAME}"
        }
    }
}