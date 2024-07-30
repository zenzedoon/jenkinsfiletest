#!/usr/bin/env groovy
//déclaration de la shared library build_libs
//@Library("build_libs",'iac_libs','common_libs','deployment') _
ArrayList ansible_versions = ["8","9"]
ArrayList list_environnement = ["sandbox","nprod"]
ArrayList verbosity_list = ["v","vv","vvv","vvvv"]
properties([
    parameters([
        string(name: "ARESIS_ID", description: "Definir le code Aresis:", trim: true),
        string(name: "ARESIS_NAME", description: "Definir le nom Aresis", trim: true),
        string(name: "PAYBOOK_NAME", description:"Le nom du playbook à deployer", trim: true),
        choice(name: "VERSION", description: "Version d'Ansible a utilisé:", choices: ansible_versions.join('\n')),
        string(name: "REFERENCE_GIT_PLAYBOOK", description: "Ce paramètre correspond à une branche ou un tag du repo contenant le playbook à déployer.<br /><i>Si le champ est laissé vide, le job se base sur le tag (git) correspondant à la version indiquée.</i>", trim: true),
        string(name: "REFERENCE_GIT_INVENTAIRE", description: "Ce paramètre correspond à une branche ou un tag du repo contenant l'inventaire à déployer.<br /> <i>Si le champ est laissé vide, le job se base sur le tag (git) correspondant à la version indiquée.</i> ", trim: true)
        choice(name: "ENVIRONNEMENT", description: "nom de l'environnement", choices: list_environnement.join('\n')),
        string(name: "TAG_ANSIBLE", description: "La liste des tag ansible a utilisé", trim: true),
        string(name: "VAULT", description: "Mot de passe VAULT pour le déploiement", trim: true),
        string(name: "EXTRA_VARS", description: "Extra vars ansible à fournir au format YAML", trim: true),
        choice(name: "VERBOSITE", description: "Niveau de verbosité", choices: verbosity_list.join('\n')),
        booleanParam(name: "MODE_CHECK", description: "Si la case est cochée, le playbook sera execute en mode "dry-run"", defaultValue: false),
        booleanParam(name: "ROLLBACK", description: "Si case cochée, le playbook joué sera le suivant: XXXX_playbook_rollback.yml", defaultValue: false),
        string(name: "LIMITE", description: "Liste des limites (ansible) séparées par des virgules. ex: host1,host2,host3", trim: true)


    ])
])

def idAresis_p = params.ARESIS_ID //"04325"
def nomAresis_p = params.ARESIS_NAME //"azure_rm_sql_role_commun"
def nomPlaybook_p = parms.PAYBOOK_NAME //"playbook.yml"
def versionAnsible_p = params.VERSION
def urlInventory_p = parmas.REFERENCE_GIT_INVENTAIRE //"https://gitlab-repo-gpf.apps.eul.sncf.fr/dosn/groupefabit-dosn/04325/azure_rm_sql_role_commun.git"
def urlPlaybook_p = params.REFERENCE_GIT_PLAYBOOK //"https://gitlab-repo-gpf.apps.eul.sncf.fr/dosn/groupefabit-dosn/04325/azure_rm_sql_role_commun.git"
def shallowClone_p = true
def options_p = ""
def version_p = params.VERSION
def reference_git_playbook_p = params.REFERENCE_GIT_PLAYBOOK
def reference_git_inventaire_p = params.REFERENCE_GIT_INVENTAIRE
def env_p = params.ENVIRONNEMENT
def tags_p = params.TAG_ANSIBLE
def vault_p = params.VAULT
def extra_p = params.EXTRA_VARS
def verbosity_p = params.VERBOSITE
def checkString_p = params.MODE_CHECK
def rollback_p = params.ROLLBACK
def limite_p = params.LIMITE
boolean success = false

//withCredentials([
//    usernamePassword(credentialsId: 'KPI', passwordVariable: 'LOGS_PASS', usernameVariable: 'LOGS_USER'),
//    usernameColonPassword(credentialsId: 'pic-dosn_gitlab-repo-gpf', variable: 'CREDENTIAL_GIT')
//    ]) {
withTools(extraWorkspaceSizeGi: 64, [[image:'04325/ansible-bdd-tools-full', name: 'ansible-full', version: '1.0.0', registry: 'sncf'],[name: 'sonar-scanner', version: 'latest', registry: 'eul']], timeout: 180) { 
    try{
        stage('Checkout') {
                    println "🔰 Récupération du code source de la branche $BRANCH_NAME"
                    scmInfo = checkout scm
                    env.GIT_URL = scmInfo.GIT_URL
                    env.GIT_COMMIT = scmInfo.GIT_COMMIT
                    println "✔️ Récupération du code source depuis $GIT_URL effectuée"
                }
        stage('Setup') {
                    container('sonar-scanner') {
                        println '🔰 Configuration du build'
                        // Mise à jour d'ansible-lint avec la version la plus récente
                        sh 'pip3 install ansible-lint'
                        println '✔️️ Configuration du build effectuée'
                    }
                }

        stage('QA_Test') {
            println '🔰 Analyse qualité'
                        // Affichage de la version d'ansible-lint, à titre informatif
                        sh 'ansible-lint --version'
                        // Exécution d'ansible-lint
                        // L'option -p permet la remontée dans Jenkins des informations relatives aux résultats d'ansible-lint
                        if (sh(script: 'ansible-lint . -p', returnStatus: true) != 0) {
                            error '❌ ansible-lint a identifié une ou plusieurs erreurs fatales'
                        }
                        gitlabCommitStatus('sonarqube') {
                            withSonarQubeEnv('sonarqube') {
                                String sonarProjectName = "${options['PROJECT_NAME']}"
                                String branchName = env.BRANCH_NAME?.replaceAll(/[\\]/, '_').replaceAll(/[, ]/, '')
                                sonarProjectName += " ${branchName}"
                                // La version d'un projet Ansible est le nom de la branche ou du tag
                                String sonarProjectVersion = "${branchName}"
                                // Exécution de l'analyse SonarQube
                                sh """\
                                    sonar-scanner \
                                    -Dsonar.projectKey=${options['PROJECT_SONAR_KEY_BRANCH']} \
                                    -Dsonar.sources=. \
                                    -Dsonar.projectName="${sonarProjectName}" \
                                    -Dsonar.projectVersion=${sonarProjectVersion} \
                                    -Dsonar.links.ci=${JOB_URL} \
                                    -Dsonar.links.homepage=${GIT_URL} \
                                """.stripIndent()
                            }
                        }
                        println '✔️ Analyse qualité effectuée'
        }
        stage('ansible_deployement') {
            container(ansible-full){
                deploy {
                    idAresis = idAresis_p
                    nomAresis = nomAresis_p
                    playbookName = nomPlaybook_p
                    versionAnsible = versionAnsible_p
                    urlInventory = urlInventory_p
                    urlPlaybook = urlPlaybook_p
                    options = options_p
                    version = version_p
                    reference_git_playbook = reference_git_playbook_p
                    reference_git_inventaire = reference_git_inventaire_p
                    env = env_p
                    extra = extra_p
                    tag = tags_p
                    vault = vault_p
                    verbosity = verbosity_p
                    checkString = checkString_p
                    rollback = rollback_p
                    limite = limite_p
                    shallowClone = shallowClone_p
                    }
                if(currentBuild.result == "SUCCESS"){
                        success = true
                    }
                    else{
                        success = false
                    }
                }
        }
        catch(Exception err){
            success = false
            currentBuild.result = 'FAILURE'
            print err
        }
    
        finally{            
                if(success){
                    println '✔️️ Role ansible deployé avec succes'
                    // subject = "Role Ansible a été bien deployé"
                    // content = "Bonjour,<br/><br/> Vous trouverez ci-joint le reporting de création des machines Vsphere des dernières 24H.<br/> Le fichier a été déposé sur le serveur ugdalsup03 dans le directory /home/ansible/VM_selfservice_vsphere <br/> Cordialement,<br/><br/> L'équipe Starter-Team."
                    // emailext(
                    //     mimeType: 'text/html',
                    //     attachmentsPattern: 'deployment/reports/*.json',
                    //     body: content.toString(),
                    //     subject: subject.toString(),
                    //     to: "ext.zied.ben-salem@sncf.fr"
                    // )
                }
                else{
                    error '❌ erreur lors du deployement du role ansible'
                    // subject = "Génération reports vpshere unsuccessful"
                    // content = "Failed ${BUILD_URL}"
                    // emailext body: content, subject: subject, to: "ld.dosn.starter.team@sncf.fr"
                }
                }

   }
}
