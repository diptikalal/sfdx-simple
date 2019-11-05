#!groovy
import groovy.json.JsonSlurper 
node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH
    def defaultdevorg="my-hub-org"

    def toolbelt = tool 'toolbelt'

    stage('checkout source') {
        // when running in multi-branch job, one must issue this command
        checkout scm
    }

    withCredentials([file(credentialsId: JWT_KEY_CRED_ID, variable: 'jwt_key_file')]) {
        stage('Create Scratch Org') {

           rc = bat returnStatus: true, script: "\"${toolbelt}\" force:auth:jwt:grant  --clientid ${CONNECTED_APP_CONSUMER_KEY} --username ${HUB_ORG} --jwtkeyfile \"${jwt_key_file}\" --setdefaultdevhubusername --instanceurl ${SFDC_HOST}"
                  if (rc != 0) { error 'hub org authorization failed' }
           println(rc.getClass())
           /*del = bat returnStatus: true, script: "\"${toolbelt}\"  force:org:delete -u \"test-x8lu7eixutve@example.com\""
           println(del)*/
            
           orglist=bat returnStatus: true, script: "\"${toolbelt}\" force:org:list --json"
          
            println(orglist.getClass())
            printf orglist
        
         /*rm = bat returnStatus: true, script: "\"${toolbelt}\" force:config:set defaultdevhubusername=${HUB_ORG} --global"
    
            // need to pull out assigned username
         rmsg = bat returnStatus: true, script: "\"${toolbelt}\" force:org:create --definitionfile config/project-scratch-def.json --json --targetdevhubusername ${HUB_ORG} --setalias my-scratch-org"
         //println(rmsg.getClass())
         printf rmsg
    

           def jsonSlurper = new JsonSlurper()
            def robj = jsonSlurper.parseText(rmsg)
            if (robj.status != 0) { error 'org creation failed: ' + robj.message }
            SFDC_USERNAME=robj.result.username
            robj = null*/

        }

        stage('Push To Test Org') {
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
            if (rc != 0) {
                error 'push failed'
            }
            // assign permset
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname DreamHouse"
            
            if (rc != 0) {
                error 'permset:assign failed'
            }
        }

        stage('Run Apex Test') {
            bat "md ${RUN_ARTIFACT_DIR}"
            timeout(time: 120, unit: 'SECONDS') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
                if (rc != 0) {
                    error 'apex test run failed'
                }
            }
        }

        stage('collect results') {
            junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
        }
    }
}
