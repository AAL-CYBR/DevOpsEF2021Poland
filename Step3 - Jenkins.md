# Workshop guide: **Step 3** Use Case 2: ***Jenkins***



## Pre-Reqs

  Step 1 outcome and will include:
- Conjur Instance up and running
- Client successfully up and running
- Install Jenkins (Included in the guide)


Docker will be used to run Jenkins container


## Install Jenkins

Before installing Jenkins a few preparation steps are required. Conjur Certificate needs to be trusted on the Jenkins side.

Create a directory for the use case first:
```Bash
mkdir -p $HOME/jenkins-conjur/certs && cd $HOME/jenkins-conjur && chmod 777 certs
```

**1. Pull the image, extract the JDK Trust Store and add the Conjur Certiificate to it**
```Bash
openssl s_client -showcerts -connect localhost:8443  < /dev/null | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > certs/conjur.pem
docker pull docker.io/jenkins/jenkins:lts-jdk11
docker run --rm -t -v $PWD/certs:/tmp/certs docker.io/jenkins/jenkins:latest bash -c "/bin/cp /opt/java/openjdk/lib/security/cacerts /tmp/certs/; \
/opt/java/openjdk/bin/keytool -import -keystore /tmp/certs/cacerts -file /tmp/certs/conjur.pem -alias conjur -trustcacerts -noprompt -storepass changeit > /dev/null 2>&1"
docker run -p 8088:8080 -p 50000:50000 --add-host=proxy:YOURDOCKERHOSTIP -d -v $PWD/certs/cacerts:/opt/java/openjdk/lib/security/cacerts --name jenkins docker.io/jenkins/jenkins:lts-jdk11
```

**2. Install Jenkins and the Conjur Plugin**

Open Jenkins Web UI in a browser and wait for it to be ready
```Bash
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

Use the secret to continue the Jenkins Setup and Click Select Plugins to Install -> Click None, and Install .... create an admin and passsword
Once Jenkins is up and running,
Download the ***Conjur.hpi*** plugin file from (here)[https://github.com/cyberark/conjur-credentials-plugin/releases]
 Go to Manage -> Manage Plugins -> Advanced. In the Upload Section, Upload the Conjur.hpi file, Tick restart Jenkins ... wait for the Restart

**3. Configure Jenkins**

Navigate to Manage Jenkins -> Manage Credentials > Jenkins > Global credentials (unrestricted)

Click "Add Credentials"


| Property      | Value     
|---------------|-----------
| Kind          | Username with Password
| Username      | **host/BotApp/myDemoApp**  
| Password      | **\<BotApp API Key>**
| The Jenkins ID| Natively provided by Jenkins
| Description 	| Optional. Provide a description to identify this global credential entry.

Navigate Jenkins -> Manage Jenkins -> Configure System, and under **Conjur Appliance**

| Property      | Value     
|---------------|-----------
| Account       | myConjurAccount
| Appliance URL	| https://HOSTMACHINE:8443
| Conjur Auth Credentials |	The host name and API key to authenticate to Conjur. Select credentials previously configured, or click Add to add new values.

**4. Configure the Secret and consume it**

Navigate Jenkins -> Manage Jenkins -> Manage Credentials -> Jenkins -> Global Credentials (unrestricted) -> Add Credentials

| Property      | Value     
|---------------|-----------
| Kind 			| Conjur Secret Credential
| Scope		    | Global
| Variable Path	| **BotApp/secretVar**
| ID			| **botapp_secretvar** (An ID to use in Jenkins to reference this variable. It does not need to match the name in Conjur.)
|Description    |Optionally provide a description of this secret.

Create a neew Freestyle Project
  - On Build Environment Section, tick "se secret text(s) or file(s)" and select "Add Conjur Secret Credential"
  - Customise the Name of the Variable to suit your need.
  - Select the specific credential and select the ConjurSecret:BotApp/secretVar
  - Add an "Execute Shell build step" and add the following
```Bash
echo "Will use this secret here ${CONJUR_SECRET}" > result.txt
cat result.txt
```Bash

Run the job, the Credential should not be leaked in both calls even when enabling the Debug mode

