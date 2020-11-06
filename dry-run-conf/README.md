# The dry run configuration

In this folder, is the `settings.xml` used to externally configure the `mvn clean deploy` to the Clever Cloud Artifactory service.


In the `settings.xml`, are factorized the following configuration elements :

* the `artifactory-maven-plugin` `<plugin>` version `2.7.0`, see [in `pom.xml`](../simple-mvn-prj/pom.xml#L44) :
  * this plugin configuration configures the target artifactory server, see [in `pom.xml`](../simple-mvn-prj/pom.xml#L65)
  * this plugin configuration configures the artifactory authentifcation credentials, see [in `pom.xml`](../simple-mvn-prj/pom.xml#L66) and [in `pom.xml`](../simple-mvn-prj/pom.xml#L67)
* the `maven-jar-plugin` `<plugin>` version `latest`, see [in `pom.xml`](../simple-mvn-prj/pom.xml#L90)
* the `maven-war-plugin` `<plugin>` version `latest`, see [in `pom.xml`](../simple-mvn-prj/pom.xml#L99)
* the `https://jcenter.bintray.com` `<pluginRepository>`



```bash

```
