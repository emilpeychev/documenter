pipeline {
    agent {
        label 'docker-agent'
    }

    parameters {
        string(name: 'REPO_URL', defaultValue: '', description: 'Git repository URL to clone')
        text(name: 'PROMPT_IA', defaultValue: '', description: '''Paste your AI prompt here.
Use ${content} where the source code snippet should go.''')
        string(name: 'QUESTION', defaultValue: '', description: 'Optional question for final Q&A')
        string(name: 'BRANCH', defaultValue: 'master', description: 'Git branch to checkout')
    }

    environment {
        FastAPI = "http://langchain:8081"
    }

    stages {
        stage('Cleanup Old Workspace') {
            steps {
                echo 'Cleaning workspace...'
                deleteDir()
            }
        }

        stage('Clone Repository') {
            steps {
                checkout([$class: 'GitSCM',
                    branches: [[name: "*/${params.BRANCH}"]],
                    userRemoteConfigs: [[url: "${params.REPO_URL}", credentialsId: 'GitHubNew']]
                ])
            }
        }

        stage('Generate Documentation (Full Context)') {
            steps {
                script {
                    def files = []
                    files += findFiles(glob: '**/*.py')
                    files += findFiles(glob: '**/*.js')
                    files += findFiles(glob: '**/*.go')
                    files += findFiles(glob: '**/*.sh')
                    files += findFiles(glob: '**/*.yaml')
                    files += findFiles(glob: '**/*.yml')
                    files += findFiles(glob: '**/*.md')
                    files += findFiles(glob: '**/*Dockerfile')
                    files += findFiles(glob: '**/Jenkinsfile')
                    files += findFiles(glob: '**/requirements.txt')

                    if (files.isEmpty()) {
                        error "No source files found for documentation."
                    }

                    def safeText = ''
                    files.each { file ->
                        if (file == null || file.path == null) {
                            echo "Skipping a null or invalid file entry."
                            return
                        }

                        try {
                            def snippet = readFile(file.path).take(1000)
                            safeText += "\n\n# ${file.path}\n" + snippet
                        } catch (e) {
                            echo "Skipping file ${file.path}: ${e.message}"
                        }
                    }

                    def prompt = params.PROMPT_IA.replace('${content}', safeText).stripIndent()

                    writeFile file: 'fullcontext.json', text: groovy.json.JsonOutput.toJson([
                        question: params.QUESTION?.trim() ?: 'Generate documentation based on the repository.',
                        content: safeText
                    ])

                    def response = sh(script: """
                        curl --fail -s -X POST ${env.FastAPI}/fullcontext \
                        -H "Content-Type: application/json" \
                        --data @fullcontext.json
                    """, returnStdout: true).trim()

                    def parsed = readJSON text: response
                    def responseText = parsed?.response ?: 'Failed to generate documentation.'

                    writeFile file: 'summary.md', text: "# Project Documentation\n\n${responseText.trim()}"
                }
            }
        }

        stage('Archive Documentation') {
            steps {
                archiveArtifacts artifacts: '*.md', onlyIfSuccessful: true
            }
        }
    }

    post {
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
