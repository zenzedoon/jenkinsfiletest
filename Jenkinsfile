properties([
    parameters([
        string(name: "texte1", description: "teste 1:", trim: true),
        booleanParam(name: "isOK", description: "Siok ou pas", defaultValue: false),
        choice(name: "VERBOSITE", description: "Niveau de verbosité", choices: verbosity_list.join('\n'))
    ])
])

pipeline {
    agent {
        label 'docker'
    }
    stages {
        stage('Build') {
            agent {
                docker {
                    image 'docker:latest'
                    args '-v /var/run/docker.sock:/var/run/docker.sock'
                }
            }
            steps {
                script {
                    // Run the hello-world Docker container
                    sh 'docker run hello-world'
                    
                    // Collect and display the parameter data inside the Docker container
                    def texte1 = params.texte1
                    def isOK = params.isOK
                    def verbosite = params.VERBOSITE
                    
                    sh """
                    echo "Parameter texte1: ${texte1}"
                    echo "Parameter isOK: ${isOK}"
                    echo "Parameter VERBOSITE: ${verbosite}"
                    """
                }
            }
        }
    }
}
