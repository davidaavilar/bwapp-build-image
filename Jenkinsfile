node {
    def app

    stage('Clone repository') {
        checkout scm
    }

    stage('Build image') {
        //This builds the actual image; synonymous to docker build on the command line
        app = docker.build("compute/bwapp_demo", "--build-arg owner_email=${env.owner_email} .")
        echo app.id
    }

    stage('Scan Image and Publish to Jenkins') {
        try {
            prismaCloudScanImage ca: '', cert: '', dockerAddress: 'unix:///var/run/docker.sock', ignoreImageBuildTime: true, image: "compute/bwapp_demo", key: '', logLevel: 'debug', podmanPath: '', project: '', resultsFile: 'prisma-cloud-scan-results.json'
        } finally {
            prismaCloudPublish resultsFilePattern: 'prisma-cloud-scan-results.json'
        }
    }

    stage('Scan image with twistcli') {
        withCredentials([usernamePassword(credentialsId: 'twistlock_creds', passwordVariable: 'TL_PASS', usernameVariable: 'TL_USER')]) {
            sh 'curl -k -u $TL_USER:$TL_PASS --output ./twistcli https://$TL_CONSOLE/api/v1/util/twistcli'
            sh 'sudo chmod a+x ./twistcli'
            sh "./twistcli images scan --u $TL_USER --p $TL_PASS --address https://$TL_CONSOLE --details compute/bwapp_demo"
        }
    }

    stage('Test image') {
        //Ideally, we would run a test framework against our image.
        app.inside {
            sh 'echo "Tests passed"'
        }
    }

    stage('Push image') {
        //Finally, we'll push the image with two tags. 1st, the incremental build number from Jenkins, then 2nd, the 'latest' tag.
        try {
            docker.withRegistry('https://nexusdocker-master-demobuild.davila.demo.twistlock.com', 'twistlock_creds') {
                app.push("${env.BUILD_NUMBER}")
                app.push("latest")
            }
        }catch(error) {
            echo "1st push failed, retrying"
            retry(5) {
                docker.withRegistry('https://nexusdocker-master-demobuild.davila.demo.twistlock.com', 'twistlock_creds') {
                    app.push("${env.BUILD_NUMBER}")
                    app.push("latest")
                }
            }
        }
    }
}
