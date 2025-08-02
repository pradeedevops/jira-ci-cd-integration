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
                    JIRA_KEY = sh(script: "git log -1 --pretty=%B | grep -o 'KAN-[0-9]\\+'", returnStdout: true).trim()
                    if (!JIRA_KEY) {
                        error("❌ No Jira issue key found in commit.")
                    }
                    echo "🎯 Jira Key: ${JIRA_KEY}"

                    def response = sh(script: """
                        curl -s -u ${JIRA_USER}:${JIRA_API_TOKEN} \\
                        -X GET "${JIRA_BASE_URL}/rest/api/3/issue/${JIRA_KEY}" \\
                        -H "Accept: application/json"
                    """, returnStdout: true)

                    JIRA_STATUS = sh(script: "echo '${response}' | jq -r '.fields.status.name'", returnStdout: true).trim()
                    echo "📌 Jira Status: ${JIRA_STATUS}"

                    if (JIRA_STATUS != "IN PROGRESS") {
                        error("🚫 Jira issue not in 'IN PROGRESS'. Found: ${JIRA_STATUS}")
                    }
                }
            }
        }

        stage('Build & Deploy') {
            steps {
                script {
                    try {
                        echo "🛠️ Running CI/CD pipeline..."
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
        curl -s -u ${JIRA_USER}:${JIRA_API_TOKEN} \\
        -X GET "${JIRA_BASE_URL}/rest/api/3/issue/${JIRA_KEY}/transitions" \\
        -H "Accept: application/json"
    """, returnStdout: true)

    def transitionId = sh(script: "echo '${transitionsJson}' | jq -r '.transitions[] | select(.name==\"${toState}\") | .id'", returnStdout: true).trim()

    if (!transitionId) {
        error("❌ No transition from '${JIRA_STATUS}' → '${toState}'")
    }

    sh(script: """
        curl -s -u ${JIRA_USER}:${JIRA_API_TOKEN} \\
        -X POST "${JIRA_BASE_URL}/rest/api/3/issue/${JIRA_KEY}/transitions" \\
        -H "Content-Type: application/json" \\
        --data '{"transition":{"id":"${transitionId}"}}'
    """)
    echo "✅ Jira moved to: ${toState}"
}

