def notify(status){
	emailext (
		body: '$DEFAULT_CONTENT', 
		recipientProviders: [
			[$class: 'CulpritsRecipientProvider'],
			[$class: 'DevelopersRecipientProvider'],
			[$class: 'RequesterRecipientProvider']
		], 
		replyTo: '$DEFAULT_REPLYTO', 
		subject: '$DEFAULT_SUBJECT',
		to: '$DEFAULT_RECIPIENTS'
	)
}

def buildStep(ext) {
	sh "rm -rfv build-$ext"
	sh "mkdir -p build-$ext"
	sh "cd build-$ext && cmake -DCMAKE_TOOLCHAIN_FILE=/opt/cmake$ext .."
	sh "cd build-$ext && make -j8"

	if (!env.CHANGE_ID) {
		sh "mv build-$ext/airstrike publishing/deploy/airstrike/airstrike.$ext"
		//sh "cp publishing/amiga-spec/airstrike.info publishing/deploy/airstrike/airstrike.$ext.info"
	}
}

node {
	try{
		stage('Checkout and pull') {
			properties([pipelineTriggers([githubPush()])])
			if (env.CHANGE_ID) {
				echo 'Trying to build pull request'
			}

			checkout scm
		}
	
		if (!env.CHANGE_ID) {
			stage('Generate publishing directories') {
				sh "rm -rfv publishing/deploy/*"
				sh "mkdir -p publishing/deploy/airstrike"
			}
		}

		stage('Build WarpOS version') {
			buildStep('wos')
		}

		stage('Build AmigaOS 3.x version') {
			buildStep('68k')
		}

		stage('Deploying to stage') {
			if (env.TAG_NAME) {
				sh "echo $TAG_NAME > publishing/deploy/STABLE"
				sh "ssh $DEPLOYHOST mkdir -p public_html/downloads/releases/airstrike/$TAG_NAME"
				sh "scp publishing/deploy/airstrike/* $DEPLOYHOST:~/public_html/downloads/releases/airstrike/$TAG_NAME/"
				sh "scp publishing/deploy/STABLE $DEPLOYHOST:~/public_html/downloads/releases/airstrike/"
			} else if (env.BRANCH_NAME.equals('master')) {
				sh "date +'%Y-%m-%d %H:%M:%S' > publishing/deploy/BUILDTIME"
				sh "ssh $DEPLOYHOST mkdir -p public_html/downloads/nightly/airstrike/`date +'%Y'`/`date +'%m'`/`date +'%d'`/"
				sh "scp publishing/deploy/airstrike/* $DEPLOYHOST:~/public_html/downloads/nightly/airstrike/`date +'%Y'`/`date +'%m'`/`date +'%d'`/"
				sh "scp publishing/deploy/BUILDTIME $DEPLOYHOST:~/public_html/downloads/nightly/airstrike/"
			}
		}
	
	} catch(err) {
		currentBuild.result = 'FAILURE'
		notify('Build failed')
		throw err
	}
}
