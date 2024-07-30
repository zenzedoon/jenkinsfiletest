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
        string(name: "REFERENCE_GIT_INVENTAIRE", description: "Ce paramètre correspond à une branche ou un tag du repo contenant l'inventaire à déployer.<br /> <i>Si le champ est laissé vide, le job se base sur le tag (git) correspondant à la version indiquée.</i> ", trim: true),
        choice(name: "ENVIRONNEMENT", description: "nom de l'environnement", choices: list_environnement.join('\n')),
        string(name: "TAG_ANSIBLE", description: "La liste des tag ansible a utilisé", trim: true),
        string(name: "VAULT", description: "Mot de passe VAULT pour le déploiement", trim: true),
        string(name: "EXTRA_VARS", description: "Extra vars ansible à fournir au format YAML", trim: true),
        choice(name: "VERBOSITE", description: "Niveau de verbosité", choices: verbosity_list.join('\n')),
        booleanParam(name: "MODE_CHECK", description: "Si la case est cochée, le playbook sera execute en mode 'dry-run'", defaultValue: false),
        booleanParam(name: "ROLLBACK", description: "Si case cochée, le playbook joué sera le suivant: XXXX_playbook_rollback.yml", defaultValue: false),
        string(name: "LIMITE", description: "Liste des limites (ansible) séparées par des virgules. ex: host1,host2,host3", trim: true)


    ])
])

def idAresis_p = params.ARESIS_ID //"04325"
def nomAresis_p = params.ARESIS_NAME //"azure_rm_sql_role_commun"
def nomPlaybook_p = params.PAYBOOK_NAME //"playbook.yml"
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
//withTools(extraWorkspaceSizeGi: 64, [[image:'04325/ansible-bdd-tools-full', name: 'ansible-full', version: '1.0.0', registry: 'sncf'],[name: 'sonar-scanner', version: 'latest', registry: 'eul']], timeout: 180) { 
stage('Checkout') {
                    println "🔰 Récupération du code source de la branche $BRANCH_NAME"
                    scmInfo = checkout scm
                    env.GIT_URL = scmInfo.GIT_URL
                    env.GIT_COMMIT = scmInfo.GIT_COMMIT
                    println "✔️ Récupération du code source depuis $GIT_URL effectuée"
                }
stage('Setup') {
    agent {
        docker {
            image 'python:3.9'
            label 'docker'
            args '-v /var/run/docker.sock:/var/run/docker.sock'
            }
    }
    steps {
        sh 'python --versiopn'
        }
}

