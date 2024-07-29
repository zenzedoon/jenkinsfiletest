pipeline {
    agent any

    parameters {
        string(name: 'TARGET_ENV', defaultValue: 'staging', description: 'Target environment for deployment')
        booleanParam(name: 'RUN_TESTS', description: 'Run tests after build?')
        choice(name: 'BUILD_TYPE', choices: ['Debug', 'Release'], description: 'Type of build')
        text(name: 'CUSTOM_MESSAGE', description: 'Custom message for the build')
    }

    stages {
        stage('Build') {
            steps {
                echo "Building the project..."
                echo "Target Environment: ${params.TARGET_ENV}"
                echo "Build Type: ${params.BUILD_TYPE}"
                echo "Custom Message: ${params.CUSTOM_MESSAGE}"
            }
        }
        
        stage('Test') {
            when {
                expression { return params.RUN_TESTS }
            }
            steps {
                echo 'Running tests...'
                // Add your test commands here
            }
        }
        
        stage('Deploy') {
            steps {
                echo "Deploying to ${params.TARGET_ENV} environment..."
                // Add your deployment commands here
            }
        }
    }
    
    post {
        always {
            echo 'Cleaning up...'
            // Add any cleanup steps here
        }
    }
}
