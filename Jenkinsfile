pipeline {
    agent any
    environment {
        SLACK_CHANNEL = '#all-itsvansh'  // Slack channel
        SLACK_WEBHOOK_URL = 'https://hooks.slack.com/services/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'
    }
    stages {        
        stage('Request Approval') {
            steps {
                script {
                    // Send a message to Slack with an approval request
                    def message = sendSlackMessage()
                    // Wait for the response from Slack (approve/deny)
                    def response = input message: 'Do you approve?', parameters: [
                        [$class: 'TextParameterDefinition', name: 'Approval', defaultValue: 'approve']
                    ]
                    
                    // Check the response and continue or stop
                    if (response != 'approve') {
                        currentBuild.result = 'ABORTED'
                        error("Pipeline aborted by user.")
                    }
                }
            }
        }
        stage('Run Pipeline') {
            steps {
                script {
                    echo "Pipeline approved, running tasks."
                    // Run the actual tasks (tests, deployments, etc.)
                    // Example: sh 'deploy.sh'
                }
            }
        }
        stage('Notify Slack') {
            steps {
                script {
                    // Send the pipeline result to Slack
                    sendSlackMessage("Pipeline completed: ${currentBuild.result}")
                }
            }
        }
    }
    post {
        always {
            // Send final notification to Slack, regardless of the outcome
            sendSlackMessage("Pipeline result: ${currentBuild.result}")
        }
    }
}

def sendSlackMessage(String text = 'Pipeline triggered') {
    // Use Slack Incoming Webhook to send a message
    def slackMessage = [
        channel: env.SLACK_CHANNEL,
        text: text,
        attachments: [
            [
                fallback: "Do you approve the pipeline?",
                actions: [
                    [
                        type: 'button',
                        text: 'Approve',
                        name: 'approve',
                        value: 'approve'
                    ],
                    [
                        type: 'button',
                        text: 'Deny',
                        name: 'deny',
                        value: 'deny'
                    ]
                ]
            ]
        ]
    ]

    // Send the Slack message via webhook
    httpRequest(
        acceptType: 'APPLICATION_JSON',
        contentType: 'APPLICATION_JSON',
        url: env.SLACK_WEBHOOK_URL,
        httpMode: 'POST',
        requestBody: groovy.json.JsonOutput.toJson(slackMessage)
    )
}
