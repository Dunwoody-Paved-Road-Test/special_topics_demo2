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
                                }
                            }
                        }
                    }
                    // send mail
                    for (int a = 0; a < userDirectories.size(); a++){
                        def emailBody = """Your static html is now available at:
                        """
                        // get the filepaths for the user
                        def user = userDirectories[a]
                        def paths = userMap."${user}"
                        for (int b = 0; b < paths.size(); b++) {
                            def path = paths[b]
                            emailBody = emailBody + """http://98.240.222.112:49160/static-web/${workspace}/${path}/
                            """
                        }
                        emailext body: emailBody, subject: 'Paved-Road Auto Notification', to: user
                    }

                            // validate html
                            // def htmlFilesList = findFiles excludes: '', glob: userName + '/'
                            // def htmlFiles
                            // for (int x = 0; x < htmlFilesList.size(); x++) {
                            //     def contents = readFile htmlFilesList[x].toString()
                            //     htmlFiles = htmlFiles + '{'
                            // }
                            // //def json = '{"users": [ {"userEmail":' + emailList[i] + ', "results": [ {"filename":' + 
                            // def reqBody = """
                            //     {​​​​​​​​
                            //         "users": [
                            //             {​​​​​​​​
                            //                 "userEmail": "$emailList[i]",
                            //                 "results": [
                            //                     {​​​​​​​​
                            //                         "fileName": "hate.html",
                            //                         "results": "Error: Start tag seen without seeing a doctype first. Expected “<!DOCTYPE html>”.\nFrom line 1, column 1; to line 1, column 6\nError: Element “head” is missing a required instance of child element “title”.\nFrom line 1, column 7; to line 1, column 12\nWarning: Consider adding a “lang” attribute to the “html” start tag to declare the language of this document.\nFrom line 1, column 1; to line 1, column 6\nThere were errors.\n"
                            //                     }​​​​​​​​,
                            //                     {​​​​​​​​
                            //                         "fileName": "love.html",
                            //                         "results": "Error: Element “head” is missing a required instance of child element “title”.\nFrom line 1, column 22; to line 1, column 27\nWarning: Consider adding a “lang” attribute to the “html” start tag to declare the language of this document.\nFrom line 1, column 16; to line 1, column 21\nThere were errors.\n"
                            //                     }​​​​​​​​
                            //                 ]
                            //             }​​​​​​​​
                            //         ]
                            //     }​​​​​​​​
                            // """
                            // httpRequest acceptType: 'APPLICATION_JSON', contentType: 'APPLICATION_JSON', httpMode: 'POST', requestBody: reqBody, responseHandle: 'NONE', url: 'url', wrapAsMultipart: false
                            
                            //def contents = readFile '$userName/'
                        // }
                        // else {
                        //     break
                        // }
                    
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
