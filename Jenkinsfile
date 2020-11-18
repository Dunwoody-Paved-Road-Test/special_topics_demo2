pipeline {
    agent any

    stages {
        stage('copy to nginx') {
            steps {
                script {
                    def remote = [:]
                    remote.name = "vagrant"
                    remote.host = "192.168.56.21"
                    remote.allowAnyHosts = true
                    remote.failOnError = true

                    withCredentials([usernamePassword(credentialsId: 'vagrant', passwordVariable: 'password', usernameVariable: 'username')]) {
                    remote.user = username
                    remote.password = password
                    }

                    sshPut remote: remote, from: './', into: '/var/www/data/static-web'
                }
            }
        }
        stage('Email Notifications'){
            steps{
                script{
                    // format the workspace for the url 
                    def workspace = "$WORKSPACE"
                    workspace = workspace.split('/')
                    workspace = workspace[-1]
                    (userMap, userDirectories) = getCommitFiles() // see getCommitFiles function below

                    def fileContents
                    def htmlFiles
                    // send mail
                    for (int a = 0; a < userDirectories.size(); a++){
                        htmlFiles = """"""
                        def emailBody = """Your static html is now available at:\n"""
                        // get the filepaths for the user
                        def user = userDirectories[a]
                        def paths = userMap[user]
                        for (int b = 0; b < paths.size(); b++) {
                            def path = paths[b]
                        // create json request body
                            if (fileExists(path)) {
                                emailBody = emailBody + """http://98.240.222.112:49160/static-web/${workspace}/${path} \n"""
                                fileContents = readFile path
                                //print fileContents
                                // format the html for the json request body
                                
                                //def lines = fileContents.split('\n')
                                // def htmlString = ""
                                // for (int i = 0; i < lines.size(); i++){
                                //     // remove any " (double quotes)
                                //     def line = lines[i].replaceAll('"',"'")
                                //     htmlString = htmlString + line
                                // }
                                def htmlString = fileContents.replaceAll("[\\n\\t]", "");
                                print htmlString
                                def name = path.split('/')
                                name = name[-1]
                                if (b == paths.size() - 1) {
                                    htmlFiles = htmlFiles + """{"fileName": "${name}", "fileData":"${htmlString}"}"""
                                }
                                else {
                                    htmlFiles = htmlFiles + """{"fileName": "${name}", "fileData":"${htmlString}"}""" + ""","""
                                }
                            }
                            
                            
                        }
                        def jsonBody = """
                            {
                                "users": [
                                    {
                                        "htmlFiles": [ ${htmlFiles} ],
                                        "userEmail": "${user}@dunwoody.edu"
                                    }
                                ]
                            }
                            """
                        print jsonBody
                        // validate html
                        def response = httpRequest consoleLogResponseBody: true, contentType: 'APPLICATION_JSON', httpMode: 'POST', requestBody: jsonBody, url: 'http://192.168.56.25:49160/api/validateHtmlForUsers', validResponseCodes: '100:500', wrapAsMultipart: false
                        def json = response.content
                        def result = readJSON text: json
                        // format the response for the email body
                        def validations = ""
                        for (int c = 0; c < paths.size(); c++) {
                            // parse and add the json
                            validations = validations + result.users.results.fileName[0][c] + "\n"
                            print result.users.results.fileName[0][c]
                            validations = validations + result.users.results.results[0][c] + "\n\n"
                            print result.users.results.results[0][c]
                        }
                        emailBody = emailBody + "\nValidation Results:\n\n"
                        emailBody = emailBody + validations
                        emailext body: emailBody, subject: 'Paved-Road Auto Notification', to: user
                    }
                }
            }
        }
        stage('Write Changelog'){
            steps{
                script{
                    def changelogString = gitChangelog returnType: 'STRING',
                        template: """{{#commits}}{{messageTitle}},{{authorEmailAddress}},{{commitTime}}
                        {{/commits}}"""
                    writeFile file: 'ChangeLog.txt', text: changelogString
                }
            }
        }
    }
}
@NonCPS
def getCommitFiles() {
    // get the changed files in the current build
    def userDirectories = []
    def userMap = [:]
    def changeLogSets = currentBuild.changeSets
    for (int i = 0; i < changeLogSets.size(); i++) {
        def entries = changeLogSets[i].items
        for (int j = 0; j < entries.length; j++) {
            def entry = entries[j]
            //echo "${entry.commitId} by ${entry.author} on ${new Date(entry.timestamp)}: ${entry.msg}"
            def files = new ArrayList(entry.affectedFiles)
            for (int k = 0; k < files.size(); k++) {
                def file = files[k]
                echo "File Changed: ${file.path}"
                // get the user's directory and the changed file
                def filepath = "${file.path}"
                def userDir = filepath.split('/')
                userDir = userDir[0]
                
                // Map filepaths to the associated user
                if (userDir == 'Jenkinsfile') {
                    echo "skip Jenkinsfile"
                    continue
                }
                if (userMap.containsKey(userDir)) {
                    echo "user already exists"
                    userMap[userDir].add(filepath)
                    print userMap
                }
                else {
                    echo "new user, adding key"
                    userMap[userDir] = []
                    userMap[userDir].add(filepath)
                    print userMap
                    userDirectories = userDirectories + [userDir]
                    print userDirectories
                }
            }
        }
    }
    return [userMap, userDirectories]
}