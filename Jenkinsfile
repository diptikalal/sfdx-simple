#!groovy

import groovy.json.JsonSlurperClassic

node {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="tests/${BUILD_NUMBER}"
    def SFDC_USERNAME

    def str1=env.cmd_list_org
    def str2=env.cmd_create_org
    
    def HUB_ORG=env.HUB_ORG_DH
    def SFDC_HOST = env.SFDC_HOST_DH
    def JWT_KEY_CRED_ID = env.JWT_CRED_ID_DH
    def CONNECTED_APP_CONSUMER_KEY=env.CONNECTED_APP_CONSUMER_KEY_DH

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
            
            //to list orgs
            list = bat returnStdout: true, script: "\"${toolbelt}\" force:org:list --json"
            println(list)
            //println(list.getClass())
            /*ajson=list-str1
            
            def jsonSlurper = new JsonSlurperClassic()
            def robj = jsonSlurper.parseText(ajson)
            if (robj.status != 0) { error ' failed:'}
            println(robj.result.nonScratchOrgs)
            robj=null*/
            
            
            println("hello")
            //to set the defaultdev hub username
            //rm = bat returnStatus: true, script: "\"${toolbelt}\" force:config:set defaultdevhubusername=${HUB_ORG} --global"
            /*
            // to create the scratch org
            rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:org:create --definitionfile config/project-scratch-def.json --json --targetdevhubusername ${HUB_ORG} --setalias my-scratch-org"
            bjson=rmsg-str2
            println(bjson)
            def jsonSlurper = new JsonSlurperClassic()
            def extstr = jsonSlurper.parseText(bjson)
            if (extstr.status != 0) { error ' failed:'}
            println(extstr.result.username)
            SFDC_USERNAME=extstr.result.username
            extstr = null*/
        }

        stage('Push To Test Org') {
            SFDC_USERNAME="test-pporb5tder72@example.com"
           // SFDC_USERNAME="test-dg82n9rshd96@example.com"
            /*rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${SFDC_USERNAME}"
            if (rc != 0) {
                error 'push failed'
            }*/
            
            rp = bat returnStatus: true, script: "\"${toolbelt}\" sfdx force:org:open"
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
