# Gravitee dry run deployment to artifactory prototype

A simple Repo, to design the automation recipe to `mvn clean deploy` all Gravitee maven projects to a dry run maven central repository service (artifactory).



* initialize source code :

```bash
git clone git@github.com:gravitee-lab/clever-cloud-artifactory-deployment-prototype.git
cd ~/clever-cloud-artifactory-deployment-prototype/ && atom .
git flow init --defaults
export FEATURE_ALIAS="init_src_code"
git flow feature start "${FEATURE_ALIAS}"

cd ~/
rm -fr ~/workytempfolder
git clone https://github.com/gravitee-lab/jfrog-activation-guardian ~/workytempfolder
cd ~/workytempfolder
git checkout 0.0.25

cp -fR ./* ~/clever-cloud-artifactory-deployment-prototype/

rm -fr ~/clever-cloud-artifactory-deployment-prototype/simple-mvn-prj/
mkdir -p ~/clever-cloud-artifactory-deployment-prototype/simple-mvn-prj/
cp -fR ~/clever-cloud-artifactory-deployment-prototype/guard-shift/* ~/clever-cloud-artifactory-deployment-prototype/simple-mvn-prj/
rm -fr ~/clever-cloud-artifactory-deployment-prototype/guard-shift/

rm -fr ~/clever-cloud-artifactory-deployment-prototype/.circleci/
mkdir -p ~/clever-cloud-artifactory-deployment-prototype/.circleci/
cp -fR ./.circleci/* ~/clever-cloud-artifactory-deployment-prototype/.circleci/

cd ~/
rm -fr ~/workytempfolder

# ---
# -- init gravtiee component source code
git clone https://github.com/gravitee-lab/gravitee-common ~/workytempfolder
cd ~/workytempfolder
git checkout 1.18.x
rm -fr ~/clever-cloud-artifactory-deployment-prototype/gio-maven-project/
mkdir -p ~/clever-cloud-artifactory-deployment-prototype/gio-maven-project/
cp -fR ./* ~/clever-cloud-artifactory-deployment-prototype/gio-maven-project/

# ---
# and go IDE
cd ~/clever-cloud-artifactory-deployment-prototype/
atom .
export FEATURE_ALIAS="init_src_code"
export COMMIT_MESSAGE="feat.(${FEATURE_ALIAS}): adding base source code"
# git flow feature start "${FEATURE_ALIAS}"
git add --all && git commit -m "${COMMIT_MESSAGE}" && git push -u origin HEAD
# git push -u origin --all
# git flow release start
# git push -u origin --all && git push -u origin --tags
```

## maven deploy plugin

* https://maven.apache.org/guides/mini/guide-3rd-party-jars-remote.html

```bash
mvn deploy:deploy-file -DgroupId=<group-id> \
  -DartifactId=<artifact-id> \
  -Dversion=<version> \
  -Dpackaging=<type-of-packaging> \
  -Dfile=<path-to-file> \
  -DrepositoryId=<id-to-map-on-server-section-of-settings.xml> \
  -Durl=<url-of-the-repository-to-deploy>
```


* it is not possible to define the server URL with the settings.xml, see :
  * https://stackoverflow.com/questions/3298135/how-to-specify-mavens-distributionmanagement-organisation-wide
  * http://maven.40175.n5.nabble.com/Can-t-specify-distributionManagement-in-settings-xml-td3181781.html


## `Cron` Config

I want the Pipeline to run once a week, On sunday at midnight :

* The Cron configuration for that frequency is :

```ini
0 0 * * 0
```

* test config every 5 minutes: ( `*/5` _cron step syntax_ not supported by Circle CI )

```ini
*/5 * * * *
```

* test config every minute :

```ini
* * * * *
```


## Secrets Initialization

```bash
export SECRETHUB_ORG="gravitee-lab"
export SECRETHUB_REPO="cicd"
secrethub org init "${SECRETHUB_ORG}"
secrethub repo init "${SECRETHUB_ORG}/${SECRETHUB_REPO}"

# --- #
# for the DEV CI CD WorkFlow of
# the Gravitee CI CD Orchestrator
secrethub mkdir --parents "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/infra/maven/dry-run/artifactory/"

# --- #
# write quay secrets for the DEV CI CD WorkFlow of
# the Gravitee CI CD Orchestrator
export ARTIFACTORY_BOT_USER_NAME="graviteebot"
export ARTIFACTORY_BOT_USER_PWD="inyourdreams;)"
export ARTIFACTORY_REPO_RELEASE_URL="http://odbxikk7vo-artifactory.services.clever-cloud.com/dry-run-releases/"
export ARTIFACTORY_REPO_SNAPSHOTS_URL="http://odbxikk7vo-artifactory.services.clever-cloud.com/dry-run-snapshots/"

echo "ARTIFACTORY_REPO_SNAPSHOTS_URL=[${ARTIFACTORY_REPO_SNAPSHOTS_URL}]"
echo "ARTIFACTORY_REPO_RELEASE_URL=[${ARTIFACTORY_REPO_RELEASE_URL}]"


echo "${ARTIFACTORY_BOT_USER_NAME}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/infra/maven/dry-run/artifactory/user-name"
echo "${ARTIFACTORY_BOT_USER_PWD}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/infra/maven/dry-run/artifactory/user-pwd"
echo "${ARTIFACTORY_REPO_SNAPSHOTS_URL}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/infra/maven/dry-run/artifactory/snaphots-repo-url"
echo "${ARTIFACTORY_REPO_RELEASE_URL}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/infra/maven/dry-run/artifactory/release-repo-url"


# From the latest secrets, create the secret settings.xml file
export SECRETHUB_ORG="gravitee-lab"
export SECRETHUB_REPO="cicd"
export ARTIFACTORY_BOT_USER_NAME=$(secrethub read "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/infra/maven/dry-run/artifactory/user-name")
export ARTIFACTORY_BOT_USER_PWD=$(secrethub read "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/infra/maven/dry-run/artifactory/user-pwd")
export ARTIFACTORY_REPO_SNAPSHOTS_URL=$(secrethub read "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/infra/maven/dry-run/artifactory/snaphots-repo-url")
export ARTIFACTORY_REPO_RELEASE_URL=$(secrethub read "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/infra/maven/dry-run/artifactory/release-repo-url")

if [ -f ./.secret.settings.xml ]; then
  rm ./.secret.settings.xml
fi;

cat <<EOF >>./.secret.settings.xml
<?xml version="1.0" encoding="UTF-8"?>
<!--

    Copyright (C) 2015 The Gravitee team (http://gravitee.io)

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

            http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

-->
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
  <pluginGroups></pluginGroups>
  <proxies></proxies>

  <servers>
    <server>
      <id>clever-cloud-artifactory-dry-run-releases</id>
      <username>${ARTIFACTORY_BOT_USER_NAME}</username>
      <password>${ARTIFACTORY_BOT_USER_PWD}</password>
    </server>
    <server>
      <id>clever-cloud-artifactory-releases</id>
      <username>${ARTIFACTORY_BOT_USER_NAME}</username>
      <password>${ARTIFACTORY_BOT_USER_PWD}</password>
    </server>
  </servers>
  <profiles>
    <profile>
      <id>gravitee-dry-run</id>
        <properties>
          <altDeploymentRepository>clever-cloud-artifactory-dry-run-releases::default::${ARTIFACTORY_REPO_RELEASE_URL}</altDeploymentRepository>
        </properties>
        <activation>
            <property>
                <name>performRelease</name>
                <value>true</value>
            </property>
        </activation>
    </profile>
    <profile>
      <id>gravitee-release</id>
        <properties>
          <altDeploymentRepository>clever-cloud-artifactory-dry-run-releases::default::${ARTIFACTORY_REPO_RELEASE_URL}</altDeploymentRepository>
        </properties>
        <activation>
            <property>
                <name>performRelease</name>
                <value>true</value>
            </property>
        </activation>
    </profile>
  </profiles>
  <activeProfiles>
  <activeProfile>gravitee-release</activeProfile>
  <activeProfile>gravitee-dry-run</activeProfile>
  </activeProfiles>
</settings>
EOF

# secrethub write --in-file ./.secret.settings.xml "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/infra/maven/dry-run/artifactory/settings.xml"
secrethub write --in-file ./.secret.settings.xml "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/infra/maven/dry-run/artifactory/settings.xml"
secrethub read --out-file ./test.retrievieving.settings.xml "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/infra/maven/dry-run/artifactory/settings.xml"

cat ./test.retrievieving.settings.xml

rm ./test.retrievieving.settings.xml

exit 0

# --- # --- # --- # --- # --- # --- # --- # --- # --- #
# --- # --- # --- # --- # --- # --- # --- # --- # --- #
# --- # --- # --- # --- # --- # --- # --- # --- # --- #
#        GPG Key Pair of the Gravitee Lab Bot         #
#                for Github SSH Service               #
#                to GPG sign maven artifacts          #
#        >>> GPG version 2.x ONLY!!!                  #
# --- # --- # --- # --- # --- # --- # --- # --- # --- #
# --- # --- # --- # --- # --- # --- # --- # --- # --- #
# --- # --- # --- # --- # --- # --- # --- # --- # --- #
# -------------------------------------------------------------- #
# -------------------------------------------------------------- #
# for the Gravitee CI CD Bot in
# the https://github.com/gravitee-lab Github Org
# -------------------------------------------------------------- #
# -------------------------------------------------------------- #
# https://www.gnupg.org/documentation/manuals/gnupg-devel/Unattended-GPG-key-generation.html
export GRAVITEEBOT_GPG_USER_NAME="Gravitee.io Lab Bot"
export GRAVITEEBOT_GPG_USER_NAME_COMMENT="Gravitee CI CD Bot in the https://github.com/gravitee-lab Github Org"
export GRAVITEEBOT_GPG_USER_EMAIL="contact@gravitee-lab.io"
export GRAVITEEBOT_GPG_PASSPHRASE="th3gr@vit331sdab@s3"

echo "Creating a GPG KEY for the Gravitee.io bot with username [${GRAVITEEBOT_GPG_USER_NAME}] and email [${GRAVITEEBOT_GPG_USER_EMAIL}], then hit the enter Key to proceed secrets initalization"
echo "# -----------------------------------"
# https://www.gnupg.org/documentation/manuals/gnupg-devel/Unattended-GPG-key-generation.html
export GNUPGHOME="$(mktemp -d)"
cat >./gravitee-lab-cicd-bot.gpg <<EOF
%echo Generating a basic OpenPGP key
Key-Type: RSA
Key-Length: 4096
Subkey-Type: RSA
Subkey-Length: 4096
Name-Real: ${GRAVITEEBOT_GPG_USER_NAME}
Name-Comment: ${GRAVITEEBOT_GPG_USER_NAME_COMMENT}
Name-Email: ${GRAVITEEBOT_GPG_USER_EMAIL}
Expire-Date: 0
Passphrase: ${GRAVITEEBOT_GPG_PASSPHRASE}
# Do a commit here, so that we can later print "done" :-)
%commit
%echo done
EOF
gpg --batch --generate-key ./gravitee-lab-cicd-bot.gpg
echo "GNUPGHOME=[${GNUPGHOME}] remove that directory when finished initializing secrets"
ls -allh ${GNUPGHOME}
gpg --list-secret-keys
gpg --list-keys

export GRAVITEEBOT_GPG_SIGNING_KEY=$(gpg --list-signatures -a "${GRAVITEEBOT_GPG_USER_NAME} (${GRAVITEEBOT_GPG_USER_NAME_COMMENT}) <${GRAVITEEBOT_GPG_USER_EMAIL}>" | grep 'sig' | tail -n 1 | awk '{print $2}')
echo "GRAVITEEBOT - GPG_SIGNING_KEY=[${GRAVITEEBOT_GPG_SIGNING_KEY}]"

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++ #
# ------------------------------------------------------------------------------------------------ #
# -- SAVING SECRETS TO SECRETHUB --                                                   -- SECRET -- #
# ------------------------------------------------------------------------------------------------ #
echo "To verify the GPG signature \"Somewhere else\" we will also need the GPG Public key"
export GPG_PUB_KEY_FILE="$(pwd)/graviteebot.gpg.pub.key"
export GPG_PRIVATE_KEY_FILE="$(pwd)/graviteebot.gpg.priv.key"

# --- #
# saving public and private GPG Keys to files
gpg --export -a "${GRAVITEEBOT_GPG_USER_NAME} (${GRAVITEEBOT_GPG_USER_NAME_COMMENT}) <${GRAVITEEBOT_GPG_USER_EMAIL}>" | tee ${GPG_PUB_KEY_FILE}
# gpg --export -a "Jean-Baptiste Lasselle <jean.baptiste.lasselle.pegasus@gmail.com>" | tee ${GPG_PUB_KEY_FILE}
# -- #
# Will be interactive for private key : you
# will have to type your GPG password
gpg --export-secret-key -a "${GRAVITEEBOT_GPG_USER_NAME} (${GRAVITEEBOT_GPG_USER_NAME_COMMENT}) <${GRAVITEEBOT_GPG_USER_EMAIL}>" | tee ${GPG_PRIVATE_KEY_FILE}
# gpg --export-secret-key -a "Jean-Baptiste Lasselle <jean.baptiste.lasselle.pegasus@gmail.com>" | tee ${GPG_PRIVATE_KEY_FILE}



export SECRETHUB_ORG="gravitee-lab"
export SECRETHUB_REPO="cicd"
secrethub mkdir --parents "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg"

export GRAVITEEBOT_GPG_USER_NAME="Gravitee.io Lab Bot"
export GRAVITEEBOT_GPG_USER_NAME_COMMENT="Gravitee CI CD Bot in the https://github.com/gravitee-lab Github Org"
export GRAVITEEBOT_GPG_USER_EMAIL="contact@gravitee-lab.io"
export GRAVITEEBOT_GPG_PASSPHRASE="th3gr@vit331sdab@s3"


echo "${GRAVITEEBOT_GPG_USER_NAME}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/user_name"
echo "${GRAVITEEBOT_GPG_USER_NAME_COMMENT}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/user_name_comment"
echo "${GRAVITEEBOT_GPG_USER_EMAIL}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/user_email"
echo "${GRAVITEEBOT_GPG_PASSPHRASE}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/passphrase"
echo "${GRAVITEEBOT_GPG_SIGNING_KEY}" | secrethub write "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/key_id"
secrethub write --in-file ${GPG_PUB_KEY_FILE} "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/pub_key"
secrethub write --in-file ${GPG_PRIVATE_KEY_FILE} "${SECRETHUB_ORG}/${SECRETHUB_REPO}/graviteebot/gpg/private_key"


# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# -- TESTS --   Testing using the new GPG Key : sign a file, and verify file signature -- TESTS -- #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #
# ------------------------------------------------------------------------------------------------ #

# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++ #
# ------------------------------------------------------------------------------------------------ #
# -- TESTS --                          First Let's Sign a file                         -- TESTS -- #
# ------------------------------------------------------------------------------------------------ #
cat >./some-file-to-sign.txt <<EOF
Hey I ma sooo important a file that
I am in a file which is going to be signed to proove my integrity
EOF

export GRAVITEEBOT_GPG_PASSPHRASE="your gpg passphrase to test all that on your computer"



# echo "${GRAVITEEBOT_GPG_PASSPHRASE}" | gpg --pinentry-mode loopback --passphrase-fd 0 --sign ./some-file-to-sign.txt

# ---
# That's Jean-Baptiste Lasselle's GPG_SIGNING_KEY for git (used as example)
export GPG_SIGNING_KEY_ID=7B19A8E1574C2883
# ---
# That's the GPG_SIGNING_KEY used buy the "Gravitee.io Lab Bot" for git and signing any file
export GRAVITEEBOT_GPG_SIGNING_KEY=$(gpg --list-signatures -a "${GRAVITEEBOT_GPG_USER_NAME} (${GRAVITEEBOT_GPG_USER_NAME_COMMENT}) <${GRAVITEEBOT_GPG_USER_EMAIL}>" | grep 'sig' | tail -n 1 | awk '{print $2}')
echo "GRAVITEEBOT - GPG_SIGNING_KEY=[${GRAVITEEBOT_GPG_SIGNING_KEY}]"

gpg --keyid-format LONG -k "0x${GRAVITEEBOT_GPG_SIGNING_KEY}"

echo "${GRAVITEEBOT_GPG_PASSPHRASE}" | gpg -u "0x${GRAVITEEBOT_GPG_SIGNING_KEY}" --pinentry-mode loopback --passphrase-fd 0 --sign ./some-file-to-sign.txt
echo "${GRAVITEEBOT_GPG_PASSPHRASE}" | gpg -u "0x${GRAVITEEBOT_GPG_SIGNING_KEY}" --pinentry-mode loopback --passphrase-fd 0 --detach-sign ./some-file-to-sign.txt



# -- #
# Will be interactive for private key : you
# will have to type your GPG password
# gpg --export-secret-key -a "${GRAVITEEBOT_GPG_USER_NAME} <${GRAVITEEBOT_GPG_USER_EMAIL}>" | tee ${GPG_PRIVATE_KEY_FILE}

# ------------------------------------------------------------------------------------------------ #
# # - To sign a GPG Key  with 1 specific private keys
# gpg --local-user 0xDEADBEE5 --sign file
# # - To sign a GPG Key  with 2 private keys
# gpg --local-user 0xDEADBEE5 --local-user 0x12345678 --sign file
# # - To sign a GPG Key  with 1 specific private keys
# gpg -u 0xDEADBEE5 --sign file
# # - To sign a GPG Key  with 2 private keys
# gpg -u 0xDEADBEE5 --local-user 0x12345678 --sign file
# ------------------------------------------------------------------------------------------------ #
echo "# ------------------------------------------------------------------------------------------------ #"
echo "the [$(pwd)/some-file-to-sign.txt] file is the file which was signed"
ls -allh ./some-file-to-sign.txt
echo "the [$(pwd)/some-file-to-sign.txt.sig] file is the signed file which was signed, and has its signature embedded"
ls -allh ./some-file-to-sign.txt.gpg
echo "the [$(pwd)/some-file-to-sign.txt.sig] file is the (detached) signature of the file which was signed"
ls -allh ./some-file-to-sign.txt.sig
echo "# ------------------------------------------------------------------------------------------------ #"
echo "In software, we use detached signatures, because when you sign a very "
echo "big size file, distributing the signature does not force distributing a very big file"
echo "# ------------------------------------------------------------------------------------------------ #"







# ++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++ #
# ------------------------------------------------------------------------------------------------ #
# -- TESTS --   Now test verifying the signed file, using its detached signature       -- TESTS -- #
# ------------------------------------------------------------------------------------------------ #

echo "  Now testing verifying the file with its detached signature :"
gpg --verify ./some-file-to-sign.txt.sig some-file-to-sign.txt
echo "# ------------------------------------------------------------------------------------------------ #"
echo "  Now testing verifying the file with its detached signature, in another Ephemeral GPG Keyring "
echo "# ------------------------------------------------------------------------------------------------ #"
export EPHEMERAL_KEYRING_FOLDER=$(mktemp -d)
chmod 700 ${EPHEMERAL_KEYRING_FOLDER}
export GNUPGHOME=${EPHEMERAL_KEYRING_FOLDER}
# gpg --list-secret-keys
# gpg --list-pub-keys
gpg --list-keys
echo "# ------------------------------------------------------------------------------------------------ #"
unset GNUPGHOME
echo "  First, without resetting GNUPGHOME env. var.  "
echo "  (we are still in the default Keyring for the current Linux User, so verifying will be successful) "
echo "# ------------------------------------------------------------------------------------------------ #"
gpg --verify ./some-file-to-sign.txt.sig some-file-to-sign.txt
echo "# ------------------------------------------------------------------------------------------------ #"
echo "  Now let's switch to the created Ephemeral GPG Keyring (Ephemeral GPG Context)"
echo "# ------------------------------------------------------------------------------------------------ #"
export GNUPGHOME=${EPHEMERAL_KEYRING_FOLDER}
# gpg --list-secret-keys
# gpg --list-pub-keys
gpg --list-keys
echo "# ------------------------------------------------------------------------------------------------ #"
echo "  Ok, there is no GPG Public key in this Ephemral GPG context"
echo "  That's why verifying the signed file with its detached signature, will fail : "
echo "    => a GPG signature is \"bound\" to its associated Public Key "
echo "    => GPG signature is Asymetric Cryptography (very important)"
echo "# ------------------------------------------------------------------------------------------------ #"
gpg --verify ./some-file-to-sign.txt.sig some-file-to-sign.txt

# now we import the Public Key in the Ephemeral Context, trust it ultimately, and verify the file signature again
gpg --import "${GPG_PUB_KEY_FILE}"
# now we trust ultimately the Public Key in the Ephemeral Context,
export GPG_SIGNING_KEY_ID=$(gpg --list-signatures -a "${GRAVITEEBOT_GPG_USER_NAME} <${GRAVITEEBOT_GPG_USER_EMAIL}>" | grep 'sig' | tail -n 1 | awk '{print $2}')
echo "GPG_SIGNING_KEY_ID=[${GPG_SIGNING_KEY_ID}]"

echo -e "5\ny\n" |  gpg --command-fd 0 --expert --edit-key ${GPG_SIGNING_KEY_ID} trust

# And verify the file signature again
gpg --verify ./some-file-to-sign.txt.sig some-file-to-sign.txt


```


* creating secrethub service account with permissions to access secrets in `gravitee-lab/cicd-infra` repo :

```bash
export NAME_OF_REPO_IN_ORG="gravitee-lab/cicd-infra"
secrethub service init "${NAME_OF_REPO_IN_ORG}" --description "Circle CI Service for Gravitee CI CD Orchestrator" --permission read | tee ./.the-created.service.token
```
* Then created a Circle CI Org context `cicd-infra`, and in that context, the `SECRETHUB_CREDENTIAL` env. var. with value, the token in output of the `service init` command





## Packaging and deployment to JFrog


Here is a sample script of what the Circle CI Pipeline basically executes :

```bash

# commandes maven

export GUARDIAN_GIT_URI="https://github.com/gravitee-lab/gravitee-gateway"
mkdir -p "$PWD/guard-shift"
git clone "${GUARDIAN_GIT_URI}" $PWD/guard-shift
cd ${MVN_LAB}

# mapping [ -v "$PWD/target:/usr/src/mymaven/target" ] requires to create a docker image to manage UID GID of linux user inside and outside container
# docker run -it --rm -v "$PWD":/usr/src/mymaven -v "$HOME/.m2":/root/.m2 -v "$PWD/target:/usr/src/mymaven/target" -w /usr/src/mymaven maven mvn clean package

# To run mvn clean package:
# [-v "$PWD":/usr/src/mymaven] :  maps the source code inside container
# [-v "$HOME/.m2":/root/.m2] : maps the maven [.m2] on my workstation to the one inside the container. I will ust this one to use settings.xml
# docker run -it --rm -v "$PWD":/usr/src/mymaven -v "$HOME/.m2":/root/.m2 -w /usr/src/mymaven maven mvn clean package
# To run mvn clean package release :
# all gravtiee java pom projects are [https://github.com/gravitee-io/gravitee-parent/]

export JFROG_BUILD_NUMBER='158468578'
#
# see https://github.com/jfrog/project-examples/tree/master/artifactory-maven-plugin-example
export JFROG_USERNAME_DEV=$(secrethub read gravitee-lab/cicd-infra/dev/jfrog/username)
export JFROG_SECRET_DEV=$(secrethub read gravitee-lab/cicd-infra/dev/jfrog/password)
echo "JFROG_USERNAME_DEV=[${JFROG_USERNAME_DEV}]"
echo "JFROG_BUILD_NUMBER=[${JFROG_BUILD_NUMBER}]"

# echo "JFROG_SECRET_DEV=[${JFROG_SECRET_DEV}]"

export DESIRED_MAVEN_VERSION=3.6.3
export MVN_DOCKER_IMAGE="maven:${DESIRED_MAVEN_VERSION}-openjdk-16 "

echo "Run Maven Clean install to package gravitee gateway from existing maven central repo (Nexus Sonatype, Maven Central)"
export MAVEN_COMMAND="mvn clean install"
echo "MAVEN_COMMAND=[${MAVEN_COMMAND}]"

docker run -it --rm -v "$PWD/guard-shift":/usr/src/mymaven -v "$HOME/.m2":/root/.m2 -w /usr/src/mymaven ${MVN_DOCKER_IMAGE} ${MAVEN_COMMAND}

echo "Run Maven Deploy using JFrog Maven Plugin"
export MAVEN_COMMAND="mvn deploy -Dusername=${JFROG_USERNAME} -Dpassword=${JFROG_SECRET} -Dbuildnumber=${JFROG_BUILD_NUMBER}"
echo "MAVEN_COMMAND=[${MAVEN_COMMAND}]"
docker run -it --rm -v "$PWD/guard-shift":/usr/src/mymaven -v "$HOME/.m2":/root/.m2 -w /usr/src/mymaven ${MVN_DOCKER_IMAGE} ${MAVEN_COMMAND}

```


# JFrog deploy plugin


The example described in this section configures the Artifactory publisher to deploy build artifacts either to the `releases` or the `snapshots` repository of the  https://gravitee.jfrog.io <!-- public OSS --> instance of Artifactory when `mvn deploy` is executed.

```Xml
<build>
    <plugins>
        ...
        <plugin>
            <groupId>org.jfrog.buildinfo</groupId>
            <artifactId>artifactory-maven-plugin</artifactId>
            <version>2.7.0</version>
            <inherited>false</inherited>
            <executions>
                <execution>
                    <id>build-info</id>
                    <goals>
                        <goal>publish</goal>
                    </goals>
                    <configuration>
                        <deployProperties>
                            <gradle>awesome</gradle>
                            <review.team>devops</review.team>
                        </deployProperties>
                        <publisher>
                            <!--<contextUrl>https://oss.jfrog.org</contextUrl>-->
                            <contextUrl>https://gravitee.jfrog.io</contextUrl>
                            <username>someUsername</username>
                            <password>somePassword</password>
                            <repoKey>libs-release-local</repoKey>
                            <snapshotRepoKey>libs-snapshot-local</snapshotRepoKey>
                        </publisher>
                    </configuration>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

When maven deploy successfully uploaded the maven artifacts to artifactory, we can see them in the artifactory Web UI :

![success mvn deploy artifacts in Artifactory Web UI](./doc/images/MVN_DEPLOY_SUCCESS_ARTIFACTORY_2020-09-27T19-10-22.862Z.png)
