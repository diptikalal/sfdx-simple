#!groovy

import groovy.json.JsonSlurperClassic

node('master') {

    def BUILD_NUMBER=env.BUILD_NUMBER
    def RUN_ARTIFACT_DIR="${BUILD_NUMBER}"
    def SFDC_USERNAME
    def var11
    def var12
    
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
            
            println(env.WORKSPACE)
            def org_create=env.WORKSPACE+str2
            println(org_create)
        
            if(env.Create_scratch_org=="Yes")
            {    
              
            //to set the defaultdev hub username
            rm = bat returnStatus: true, script: "\"${toolbelt}\" force:config:set defaultdevhubusername=${HUB_ORG} --global"
        
            // to create the scratch org
            rmsg = bat returnStdout: true, script: "\"${toolbelt}\" force:org:create --definitionfile config/project-scratch-def.json --json --targetdevhubusername ${HUB_ORG} --setalias my-scratch-org"
            bjson=rmsg-org_create
            println(bjson)
            def jsonSlurper = new JsonSlurperClassic()
            def extstr = jsonSlurper.parseText(bjson)
            if (extstr.status != 0) { error ' failed:'}
            println(extstr.result.username)
            SFDC_USERNAME=extstr.result.username
            println("sfdc_username")
            println(SFDC_USERNAME)
            //SFDC_USERNAME="mnd dipti"
           
            extstr = null
            bat "echo ${SFDC_USERNAME} > sfdc.txt"
            }
            else
            {
                println("In No part")
            }
        }

     stage('Push To Test Org') {

                bat 'dir'
                bat '''
                set /p var11=<sfdc.txt
                echo %var11%
                set var12=var11
               sfdx force:org:open --targetusername %var11%
               sfdx force:source:push --targetusername %var11%
                '''
            }
            //SFDC_USERNAME="test-1usebkhr6ilc@example.com"
           // SFDC_USERNAME="test-dg82n9rshd96@example.com"
            /*rc = bat returnStatus: true, script: "\"${toolbelt}\" force:source:push --targetusername ${refernce_var}"
            if (rc != 0) {
                error 'push failed'
            }
         
            rp = bat returnStatus: true, script: "\"${toolbelt}\" force:org:open --targetusername ${SFDC_USERNAME}"
            // assign permset
            rc = bat returnStatus: true, script: "\"${toolbelt}\" force:user:permset:assign --targetusername ${SFDC_USERNAME} --permsetname DreamHouse"
            
            if (rc != 0) {
                error 'permset:assign failed'
            }*/
        

        stage('Run Apex Test') {
            SFDC_USERNAME="test-1usebkhr6ilc@example.com"
            dir('tests')
            {
                bat "md ${RUN_ARTIFACT_DIR}"
                timeout(time: 120, unit: 'SECONDS') {
                rc = bat returnStatus: true, script: "\"${toolbelt}\" force:apex:test:run --testlevel RunLocalTests --outputdir ${RUN_ARTIFACT_DIR} --resultformat tap --targetusername ${SFDC_USERNAME}"
                if (rc != 0) {
                    error 'apex test run failed'
                }
                }
            }
        }
       stages('delete an org')
        {
            SFDC_USERNAME="test-1usebkhr6ilc@example.com"
             del = bat returnStatus: true, script: "\"${toolbelt}\" force:org:delete --targetusername ${SFDC_USERNAME} --noprompt"
        }   
            
        stage('collect results') {
            junit keepLongStdio: true, testResults: 'tests/**/*-junit.xml'
        }
    }
}
