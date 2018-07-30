stage 'CI'
node {

    checkout scm
    //git 'https://github.com/Ellison001/jenkins2-solitaire-systemjs.git'

    // pull dependencies from npm
    // on windows use: bat 'npm install'
    sh 'npm install --save-dev'

    // stash code & dependencies to expedite subsequent testing
    // and ensure same code & dependencies are used throughout the pipeline
    // stash is a temporary archive
    stash name: 'everything', 
          excludes: 'PhantomJS*/**', 
          includes: '**'
    
    // test with PhantomJS for "fast" "generic" results
    // on windows use: bat 'npm run test-single-run -- --browsers PhantomJS'
    sh 'npm run test-single-run -- --browsers PhantomJS'
    
    // archive karma test results (karma is configured to export junit xml files)
    step([$class: 'JUnitResultArchiver', 
          testResults: '**/**/unit.xml'])
          
}

node('Linux') {
    sh 'ls'
    sh 'rm -rf *'
    unstash name:'everything'
    sh 'ls'
}

stage 'Browser Testing'
parallel c: {
    //runTests("PhantomJS")
    echo "This is branch c"
}, a: {
    echo "This is branch a"
}, b: {
    //runTests("PhantomJS")
    echo "This is branch b"
}

def runTests(browser){
    node {
        sh 'rm -rf *'
        unstash name:'everything'
        sh "npm run test-single-run -- --browser ${browser}"
        step([$class: 'JUnitResultArchiver', testResults: '**/**/unit.xml'])
        
    }
}

node {
    notify("Deploy to staging?")
}

input 'Deploy to staging?'

stage name: 'Deploy to staging', concurrency: 1

node {
    sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
    sh 'sudo docker-compose up -d  --build'
    notify 'Solitaire Deployed'
}

stage name: 'Deploy to PROD', concurrency: 1

node {
    sh "echo '<h1>${env.BUILD_DISPLAY_NAME}</h1>' >> app/index.html"
    sh 'sudo docker-compose up -d  --build'
    notify 'Solitaire Deployed'
}

def notify(status){
    emailext (
      to: "ellison.zhang@activenetwork.com",
      subject: "${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]'",
      body: """<p>${status}: Job '${env.JOB_NAME} [${env.BUILD_NUMBER}]':</p>
        <p>Check console output at <a href='${env.BUILD_URL}'>${env.JOB_NAME} [${env.BUILD_NUMBER}]</a></p>""",
    )
}