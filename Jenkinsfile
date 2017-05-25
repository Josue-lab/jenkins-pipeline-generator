
node{
    def projectName         = ProjectName.trim()
    def projectTemplate     = ProjectTemplate.trim()
    def jenkins_github_id   = "${env.JENKINS_GITHUB_CREDENTIALS_ID}"
    def projectNameParsed   = ""
    def shouldApplyTemplate = projectTemplate != "no_template"
    def sbtFolder           = "${tool name: 'sbt-0.13.13', type: 'org.jvnet.hudson.plugins.SbtPluginBuilder$SbtInstallation'}/bin"
        
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
                    url: "git@github.com:telegraph/${projectNameParsed}.git"
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
            echo "Run 'sbt new ${projectTemplate}'"
            sh "${sbtFolder}/sbt new git@github.com:${projectTemplate} --name='${projectName}' -f"
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
                    ${sbtFolder}/sbt static:stackSetup
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
                        scriptPath(\'Jenkinsfile\')
                    }
                }
            }
        """
    }
}
