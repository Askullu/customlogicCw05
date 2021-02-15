#!/usr/bin/env groovy
@Library(['piper-lib', 'piper-lib-os']) _

node{
	 dockerExecuteOnKubernetes(script: this, dockerEnvVars: ['pusername':pusername, 'puserpwd':puserpwd], dockerImage: 'docker.wdf.sap.corp:51010/sfext:latest' ) {
	 stage ('Build') { 
		deleteDir()
      	checkout scm	 
	 	sh '''
			    npm config set unsafe-perm true
			    npm rm -g @sap/cds
			    npm i -g @sap/cds-dk
			    cd tests/mocks
			    cds build --production
			    cd ../..
			    mv ./build/manifest1.yml tests/mocks/gen/srv/manifest.yml
				mv ./build/xs-security.json xs-security.json
			    '''
		packageJson = readJSON file: 'package.json'
		packageJson.cds.requires.API_BUSINESS_PARTNER["[production]"].credentials.destination = "bupa-mock"
		writeJSON file: 'package.json', json: packageJson
		sh "cat package.json"
		sh "mbt build -p=cf"  
		 
	  }

	  	stage('Deploy Mock'){
			setupCommonPipelineEnvironment script:this
		 	// cloudFoundryDeploy script:this, deployTool:'cf_native', cloudFoundry: [manifest: 'tests/mocks/gen/srv/manifest.yml']
			 cloudFoundryDeploy script:this, deployTool:'mtaDeployPlugin'
		  
		  	
              }

	//    echo 'Y' | apt-get install jq    
	// 		    jq '.+{"cds": {
    // 				    "--hana": {
    //   				      "deploy-format": "hdbtable"
    //                                  },
    // 				    "requires": {
    //   				      "messaging": {
    //     				"kind": "local-messaging",
    //     				"[production]": {
    //       				  "kind": "enterprise-messaging",
    //       				  "queue": {
    //         				    "name": "refapps/bpems/abc/BusinessPartnerValidation-srv/1234"
    //       				   }
    //     				 }
    //   				       },
    //   				       "[production]": {
    //     				 "db": {
    //       				   "kind": "hana",
    //       				   "model": [
    //         				     "db",
    //         				     "srv"
    //       				    ]
    //     				   }
    //   					 },
    //   					"API_BUSINESS_PARTNER": {
    //     				  "kind": "odata",
    //     				  "model": "srv/external/API_BUSINESS_PARTNER",
    //     				  "[production]": {
    //       				    "credentials": {
    //         				      "destination": "bupa-mock",
    //         				      "requestTimeout": 18000000
    //       				     }
    //     				   }
    //  					 },
    //   					"uaa": {
    //     				 "kind": "xsuaa"
    //   					}
   	// 			      }
  	// 			    }
	// 			    }' package.json > package1.json
	// 		    mv package1.json package.json
	// 		    mv ./build/xs-security.json xs-security.json
	// 		    cat package.json
	// 		    mbt build -p=cf
	  
	//    stage('Undeploy'){
	// 	  emailext body: '$DEFAULT_CONTENT', subject: '$DEFAULT_SUBJECT', to: 'navin.krishnan.manohar@sap.com'
	//      }
}
} 

def pushToCF(cfManifest){
	cloudFoundryDeploy script:this, deployTool:'cf_native', cloudFoundry: [manifest: cfManifest]
}
