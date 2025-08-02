pipeline {
    agent any

    environment {
        JIRA_USER = 'pradeepanaveensabares@gmail.com'
        JIRA_API_TOKEN = credentials('jira-api-token')
        JIRA_BASE_URL = 'https://pradeepanaveensabares.atlassian.net'
    }

    stages {
        stage('Check Jira Status') {
            steps {
                script {
                    env.JIRA_KEY = sh(script: "git log -1 --pretty=%B | grep -o 'KAN-[0-9]\\+'", returnStdout: true).trim()
                    if (!env.JIRA_KEY) {
                        error("❌ No Jira issue key found in commit message.")
                    }
                    echo "🎯 Jira Key: ${env.JIRA_KEY}"

                    def response = sh(script: """
                        curl -s -u ${JIRA_USER}:${JIRA_API_TOKEN} \\
                        -X GET "${JIRA_BASE_URL}/rest/api/3/issue/${env.JIRA_KEY}" \\
                        -H "Accept: application/json"
                    """, returnStdout: true)

                    env.JIRA_STATUS = sh(script: "echo '${response}' | jq -r '.fields.status.name'", returnStdout: true).trim()
                    echo "📌 Jira Status: ${env.JIRA_STATUS}"

                    if (env.JIRA_STATUS.toLowerCase() != "in progress") {
                        error("🚫 Jira issue is not in 'In Progress'. Found: ${env.JIRA_STATUS}")
                    }
                }
            }
        }

        stage('Build & Deploy') {
            steps {
                script {
                    try {
                        echo "🛠️ Running CI/CD pipeline steps..."
                        sh 'echo "Build succeeded!"'
                        transitionJira('Done')
                    } catch (err) {
                        transitionJira('Blocked')
                        throw err
                    }
                }
            }
        }
    }
}

def transitionJira(String toState) {
    def transitionsJson = sh(script: """
        curl -s -u ${env.JIRA_USER}:${env.JIRA_API_TOKEN} \\
        -X GET "${env.JIRA_BASE_URL}/rest/api/3/issue/${env.JIRA_KEY}/transitions" \\
        -H "Accept: application/json"
    """, returnStdout: true)

    def transitionId = sh(script: "echo '${transitionsJson}' | jq -r '.transitions[] | select(.name | ascii_downcase == \"${toState.toLowerCase()}\") | .id'", returnStdout: true).trim()

    if (!transitionId) {
        error("❌ No valid transition from '${env.JIRA_STATUS}' → '${toState}' found.")
    }

    sh(script: """
        curl -s -u ${env.JIRA_USER}:${env.JIRA_API_TOKEN} \\
        -X POST "${env.JIRA_BASE_URL}/rest/api/3/issue/${env.JIRA_KEY}/transitions" \\
        -H "Content-Type: application/json" \\
        --data '{"transition":{"id":"${transitionId}"}}'
    """)
    echo "✅ Jira issue ${env.JIRA_KEY} transitioned to: ${toState}"
}

