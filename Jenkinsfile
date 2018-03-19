
node{

    properties ([
        parameters([
            string(name: 'GITHUB_ORGANIZATION', description: 'The organization name to be used on Github'),
            string(name: 'PROJECT_NAME', description: 'The name of the project to create'),
            choice(name: 'PROJECT_TEMPLATE', choices: 'no_template\ntelegraph/sbt-pipeline.playframework.g8', description: 'The template to use on the new project'),
            string(name: 'SBT_TOOL_NAME', defaultValue: 'sbt', description: 'The name of the Sbt tool set up on Jenkins'),
        ])
    ])

    def projectName         = PROJECT_NAME.trim()
    def projectTemplate     = PROJECT_TEMPLATE.trim()
    def jenkins_github_id   = "${env.JENKINS_GITHUB_CREDENTIALS_ID}"
    def projectNameParsed   = ""
    def shouldApplyTemplate = projectTemplate != "no_template"

    def sbt = "${tool name: SBT_TOOL_NAME, type: 'org.jvnet.hudson.plugins.SbtPluginBuilder$SbtInstallation'}/bin/sbt"

    stage("Process Project Name"){
        if(projectName.equals("")){
            error "Invalid Input Property '${ProjectName}'"
        }else{
            echo "Processing Name"
            projectNameParsed = projectName.replaceAll(/(\s+)/, "-")
            projectNameParsed = projectNameParsed.toLowerCase()
            echo "   Parsed Name: ${projectNameParsed}"
            sh "rm -rf *"
        }
    }

    stage("Checkout Project"){
        if(shouldApplyTemplate){
            echo "Checking out Project from Git"
            checkout changelog: false, poll: false, scm: [
                $class: 'GitSCM',
                branches: [[name: 'master']],
                doGenerateSubmoduleConfigurations: false,
                extensions: [[
                    $class: 'RelativeTargetDirectory',
                    relativeTargetDir: "${projectNameParsed}"
                ]],
                submoduleCfg: [],
                userRemoteConfigs: [[
                    credentialsId: "${jenkins_github_id}",
                    url: "git@github.com:${GITHUB_ORGANIZATION}/${projectNameParsed}.git"
                ]]
            ]
            dir("${projectNameParsed}") {
                sh  "git status"
                sh  "git checkout master"
                sh  "git status"
            }
        }else{
            echo "Skipping"
        }
    }

    stage("Apply SBT Template"){
        if(shouldApplyTemplate){
            echo "Run 'git clone git@github.com:${projectTemplate}'"
            sh "git clone git@github.com:${projectTemplate} sbt-template.g8"
            echo "Run 'sbt new ${projectTemplate}'"
            sh "${sbt} new file://sbt-template.g8/ --name='${projectName}' -f"
            sh "rm -rf sbt-template.g8"
        }else{
            echo "Skipping"
        }
    }

    stage("Apply Changes"){
        if(shouldApplyTemplate){
            echo "Commit & Push Changes"
            dir("${projectNameParsed}") {
                sh '''
                    git add --all;
                    git commit -m \'Merging SBT Pipeline Template\';
                    git push;
                '''
            }
        }else{
            echo "Skipping"
        }
    }
    
    stage("Create Docker Repository"){
        if(shouldApplyTemplate){
            echo "Run 'sbt static:stackSetup'"
            dir("${projectNameParsed}") {
                sh """
                    ${sbt} static:stackSetup
                """
            }
        }else{
            echo "Skipping"
        }
    }

    stage("Build Pipeline Job"){
        echo "Create Build Pipeline"
        jobDsl scriptText: """
            pipelineJob(\"Pipeline/${projectNameParsed}\"){
                displayName(\"${projectName} Pipeline\")

                environmentVariables(PROJECT_NAME: "${projectNameParsed}")

                triggers {
                    githubPush()
                }

                definition {
                    cpsScm {
                        scm {
                            git {
                                remote {
                                    github(\"telegraph/${projectNameParsed}\", \'ssh\')
                                    credentials("${jenkins_github_id}")
                                }
                                extensions {
                                    cleanBeforeCheckout()
                                    wipeOutWorkspace()
                                    submoduleOptions {
                                        disable( true )
                                    }
                                }
                                branch("master")
                            }
                        }
                        scriptPath(\'Jenkinsfile.groovy\')
                    }
                }
            }
        """
    }
}
