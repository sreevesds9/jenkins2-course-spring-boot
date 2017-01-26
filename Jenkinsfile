node {

//Using random numbers for retry wait/sleep.  
random     = new Random()
int min    = 5
int max    = 20

def project_path        = 'spring-boot-samples/spring-boot-sample-atmosphere'
def forked_github_repo  = 'https://github.com/sreevesds9/jenkins2-course-spring-boot'


try {
     
    currentBuild.result = 'SUCCESS'
    
    stage("checkout from github") {
     git "$forked_github_repo"
    }
    
    } catch (err) {
    //Sleep random range min - max
     randomInt = random.nextInt(max-min)+min
     print "Random int " + randomInt
     sleep randomInt
     retry(3) {
         git "$forked_github_repo"
     }

 }

try {
    dir("$project_path") {
        
       stage("compile") {
        sh 'mvn compile'
       }
       
       stage("build") {
        sh 'mvn clean package'
       }
    
    
      dir("target") {
        //stash jar for DEV/QA/PROD deploy
        stage("Stash Jar Artifact") {
         stash name: 'artifact', includes: '*.jar'
        }
        
        stage("Publish to AWS S3 bucket") {
           
        step([$class: 'S3BucketPublisher',
        entries: [[sourceFile: '*.jar',
        bucket: 'jenkins2',
        selectedRegion: 'us-west-2',
        noUploadOnFailure: true,
        managedArtifacts: false,
        flatten: true,
        showDirectlyInBrowser: true,
        keepForever: true]],
        profileName: 'jenkins2',
        dontWaitForConcurrentBuildCompletion: false, 
        ])
       }

      }
      
     }
    
 } catch (err) {
     stage("Email")
     notify("ERROR: ${err}")
     currentBuild.result = "Failure"
     throw err

 }

    
}

//Master or DEV agent -- edc-v-lxb101
node {
 try {
 currentBuild.result = 'SUCCESS'
  
  dir("$stage_path") {
    
      stage("Deploy to Dev") {
       unstash "artifact"
      }
     
  }
  
 } catch (err) {
     stage("Email")
     notify("ERROR: ${err}")
     currentBuild.result = "Failure"
     throw err

 }
    
}

//QA agent -- edc-v-lxb102
node('qalinux') {
    
 try {
 currentBuild.result = 'SUCCESS'
  
  dir("$stage_path") {
    
      stage("Deploy to QA") {
       unstash "artifact"
      }
     
  }
  
 } catch (err) {
     stage("Email")
     notify("ERROR: ${err}")
     currentBuild.result = "Failure"
     throw err

 }
}
 
 
 //PROD agent - prod-tcr101
 node('prodlinux') {
  try {
  currentBuild.result = 'SUCCESS'
 
  dir("$stage_path") {
    
      stage("Deploy to PROD") {
       unstash "artifact"
      }
     
  }
  
  } catch (err) {
     stage("Email Notification")
     notify("ERROR: ${err}")
     currentBuild.result = "Failure"
     throw err

  }
 }


node {
    
    stage("Email") {
        notify('Success')
    }
}
 
def notify(status) {
    
    emailext (
      to: "sreeves@epnet.com",
      subject: "${status} ${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status} Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at &QUOT;<a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a>&QUOT;</p>""",
      recipientProviders: [[$class: 'DevelopersRecipientProvider']]
    )
    
}
