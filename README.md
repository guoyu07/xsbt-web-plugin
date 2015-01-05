[![Build Status](https://travis-ci.org/earldouglas/xsbt-web-plugin.png?branch=master)](https://travis-ci.org/earldouglas/xsbt-web-plugin)
[![Codacy Badge](https://www.codacy.com/project/badge/29fa98e63b3842d9adc221d3edd67089)](https://www.codacy.com)

**Note: this is the readme for the 1.0 release.  For the older 0.9 
release, see the [0.9 branch](https://github.com/earldouglas/xsbt-web-plugin/tree/0.9).**

## About

xsbt-web-plugin is an extension to [sbt](http://www.scala-sbt.org/) for building enterprise Web applications based on the [Java J2EE Servlet specification](http://en.wikipedia.org/wiki/Java_Servlet).

xsbt-web-plugin supports both Scala and Java, and is best suited for projects that:

* Deploy to common cloud platforms (e.g. [Google App Engine](https://developers.google.com/appengine/), [Heroku](https://www.heroku.com/), [Elastic Beanstalk](https://console.aws.amazon.com/elasticbeanstalk/home), [Jelastic](http://jelastic.com/))
* Deploy to production J2EE environments (e.g. Tomcat, Jetty, GlassFish, WebSphere)
* Incorporate J2EE libraries (e.g. [JSP](http://en.wikipedia.org/wiki/JavaServer_Pages), [JSF](http://en.wikipedia.org/wiki/JavaServer_Faces), [EJB](http://en.wikipedia.org/wiki/Ejb))
* Utilize J2EE technologies (e.g. [`Servlet`](http://docs.oracle.com/javaee/6/api/javax/servlet/Servlet.html)s, [`Filter`](http://docs.oracle.com/javaee/6/api/javax/servlet/Filter.html)s, [JNDI](http://en.wikipedia.org/wiki/Java_Naming_and_Directory_Interface))
* Have a specific need to be packaged as a [*.war* file](https://en.wikipedia.org/wiki/WAR_%28Sun_file_format%29)

## Requirements

* sbt 0.13.x
* Scala 2.10.x

## Getting started 

The quickest way to get started is to clone the [xwp-template](https://github.com/earldouglas/xwp-template) 
project, which sets up the necessary directories, files, and configuration for a 
basic xsbt-web-plugin project.

There are many examples in the form of tests in [src/sbt-test](https://github.com/earldouglas/xsbt-web-plugin/tree/master/src/sbt-test).

Below is a [step-by-step guide](https://github.com/earldouglas/xsbt-web-plugin#starting-from-scratch) 
on starting a new xsbt-web-plugin project from scratch.

## How it works

xsbt-web-plugin consists of three modules: a *webapp* plugin, a *war* plugin, 
and a *container* plugin.

The *webapp* plugin is responsible for preparing a Servlet-based Web application 
as a directory, containing compiled project code, project resources, and a 
special *webapp* directory (which includes the *web.xml* configuration file, 
static HTML files, etc.).

The *war* plugin builds on the *webapp* plugin, adding a way to package the Web 
application directory as a *.war* file that can be published as an artifact, and 
deployed to a Servlet container.

The *container* plugin also builds on the *webapp* plugin, adding a way to 
launch a servlet container in a forked JVM to host the project as a Web 
application.

Put together, these compose xsb-web-plugin, and provide complete support for 
developing Servlet-based Web applications in Scala (and Java).

## Quick reference

First, add xsbt-web-plugin:

*project/plugins.sbt*:

```scala
addSbtPlugin("com.earldouglas" % "xsbt-web-plugin" % "1.0.0")
```

Then choose either Jetty or Tomcat with default setings:

*build.sbt*:

```scala
jetty()
```

*build.sbt*:

```scala
tomcat()
```

Start (or restart) the container with `container:start`:

*sbt console:*

```
> container:start
```

Stop the container with `container:stop`:

*sbt console:*

```
> container:stop
```

Build a *.war* file with `package`:

*sbt console:*

```
> package
```

### Full-configuration sbt projects

If your project uses a full *.scala*-based configuration, you'll need to use 
`com.earldouglas.xwp.XwpPlugin.jetty()` in your project settings.

Alternatively, you can use a minimal *build.sbt* that contains only `jetty()`, 
and leave the rest of your project configuration as is.

## Configuration and usage

### Triggered (re)launch

*sbt console:*

```
> ~container:start
```

This starts the container, then monitors the sources, resources, and webapp 
directories for changes, which triggers a container restart.

### Configure Jetty to run on port 9090

*build.sbt:*

```scala
jetty(port = 9090)
```

### Configure Tomcat to run on port 9090

*build.sbt:*

```scala
tomcat(port = 9090)
```

### Configure Jetty with jetty.xml

*build.sbt:*

```scala
jetty(config = "etc/jetty.xml")
```

The `config` path can be either absolute or relative to the project directory.

### Depend on libraries in a multi-project build

*build.sbt:*

```scala
lazy val root = (project in file(".")) aggregate(mylib1, mylib2, mywebapp)

lazy val mylib1 = project

lazy val mylib2 = project

lazy val mywebapp = project dependsOn (mylib1, mylib2)
```

### Add an additional source directory

*build.sbt:*

```scala
// add <project>/src/main/extra as an additional source directory
unmanagedSourceDirectories in Compile <+= (sourceDirectory in Compile)(_ / "extra")
```

Scala files in the extra source directory are compiled, and bundled in the 
project artifact *.jar* file.

### Add an additional resource directory

*build.sbt:*

```scala
// add <project>/src/main/extra as an additional resource directory
unmanagedResourceDirectories in Compile <+= (sourceDirectory in Compile)(_ / "extra")
```

Files in the extra resource directory are not compiled, and are bundled directly 
in the project artifact *.jar* file.

### Change the default Web application resources directory

*build.sbt:*

```scala
// set <project>/src/main/WebContent as the webapp resources directory
webappSrc in webapp <<= (sourceDirectory in Compile) map  { _ / "WebContent" }
```

The Web application resources directory is where static Web content (including 
*.html*, *.css*, and *.js* files, the *web.xml* container configuration file, 
etc.  By default, this is kept in *<project>/src/main/webapp*.

### Change the default Web application destination directory

*build.sbt:*

```scala
// set <project>/target/WebContent as the webapp destination directory
webappDest in webapp <<= target map  { _ / "WebContent" }
```

The Web application destination directory is where the static Web content, 
compiled Scala classes, library *.jar* files, etc. are placed.  By default, 
they go to *<project>/target/webapp*.

### Modify the contents of the prepared Web application

After the *<project>/target/webapp* directory is prepared, it can be modified 
with an arbitrary `File => Unit` function.

For example, here's how to minify JavaScript files using [YUI Compressor](https://yui.github.io/yuicompressor/):

*project/plugins.sbt*:

```scala
libraryDependencies += "com.yahoo.platform.yui" % "yuicompressor" % "2.4.7" intransitive()
```

*build.sbt:*

```scala
// minify the JavaScript file script.js to script-min.js
postProcess in webapp := {
  webappDir =>
    import java.io.File
    import com.yahoo.platform.yui.compressor.YUICompressor
    val src  = new File(webappDir, "script.js")
    val dest = new File(webappDir, "script-min.js")
    YUICompressor.main(Array(src.getPath, "-o", dest.getPath))
}
```

### Use *WEB-INF/classes* instead of *WEB-INF/lib*

By default, project classes and resources are packaged in the default *.jar* 
file artifact, which is copied to *WEB-INF/lib*.  This file can optionally be 
ignored, and the project classes and resources copied directly to 
*WEB-INF/classes*.

*build.sbt:*

```scala
webInfClasses in webapp := true
```

### Prepare the Web application for execution and deployment

For situations when the prepared *<project>/target/webapp* directory is needed, 
but the packaged *.war* file isn't.

*sbt console:*

```
webapp:prepare
```

### Use a custom webapp runner

By default, either Jetty's [jetty-runner](http://wiki.eclipse.org/Jetty/Howto/Using_Jetty_Runner) 
or Tomcat's [webapp-runner](https://github.com/jsimone/webapp-runner) will be 
used to launch the container under `container:start`.

To use a custom runner, use `runnerContainer` with `warSettings` and 
`webappSettings`:

*build.sbt:*

```scala
runnerContainer(
  libs = Seq(
      "org.eclipse.jetty" %  "jetty-webapp" % "9.1.0.v20131115" % "container"
    , "org.eclipse.jetty" %  "jetty-plus"   % "9.1.0.v20131115" % "container"
    , "test"              %% "runner"       % "0.1.0-SNAPSHOT"  % "container"
  ),
  args = Seq("runner.Run", "8080")
) ++ warSettings ++ webappSettings
```

Here, `libs` includes the `ModuleID`s of libraries needed to make our runner, 
which is invoked by calling the main method of `runner.Run` with a single 
argument to specify the server port.

### Attach a Java agent

*build.sbt:*

```scala
javaOptions += "-agentpath:/path/to/libyjpagent.jnilib"
```

### Add manifest attributes

By default, the *.war* file includes the same manifest attributes as the
project's artifact:

* Implementation-Vendor
* Implementation-Title
* Implementation-Version
* Implementation-Vendor-Id
* Specification-Vendor
* Specification-Title
* Specification-Version

These can be changed in both artifacts:

*build.sbt:*

```scala
packageOptions in (Compile, packageBin) +=
    Package.ManifestAttributes( java.util.jar.Attributes.Name.SEALED -> "true" )
```

Or in just the *.war* file:

*build.sbt:*

```scala
packageOptions in packageWar +=
  Package.ManifestAttributes( java.util.jar.Attributes.Name.SEALED -> "true" )
```

### Set forked JVM options

To specify options to be provided to the forked JVM, set `javaOptions` in the `container` task:

```scala
javaOptions in container += "-Xmx8g"
```

### Configure forking

Forking in sbt can be configured through a [`ForkOptions`](http://www.scala-sbt.org/0.13.5/api/index.html#sbt.ForkOptions) instance, by passing it as the `options` argument to the `jetty()` or `tomcat()` function:

```scala
jetty(options = new ForkOptions(runJVMOptions = Seq("-Xmx8g")))
```

The `ForkOptions` constructor takes the following arguments:

```scala
new ForkOptions(
    javaHome: Option[File] = scala.None
  , outputStrategy: Option[OutputStrategy] = scala.None
  , bootJars: Seq[File] = immutable.this.Nil
  , workingDirectory: Option[File] = scala.None
  , runJVMOptions: Seq[String] = immutable.this.Nil
  , connectInput: Boolean = false
  , envVars: Map[String, String] = ...
) 
```

### Using JRebel

1 Add the following plugin that generates *jrebel.xml* to *project/plugins.sbt*

```scala
addSbtPlugin("fi.gekkio.sbtplugins" % "sbt-jrebel-plugin" % "0.10.0")
```

2 Add the following lines to *build.sbt*, making sure to specify the correct path to JRebel

```scala
seq(jrebelSettings: _*)

jrebel.webLinks += (sourceDirectory in Compile).value / "webapp"

javaOptions in container ++= Seq(
    "-javaagent:/path/to/jrebel/jrebel.jar",
    "-noverify",
    "-XX:+UseConcMarkSweepGC",
    "-XX:+CMSClassUnloadingEnabled"
)
```

3 You can now issue 
```scala
> container:start
> ~compile
```
and your changes should be picked up automatically.

## Starting from scratch

Create a new empty project:

```
mkdir myproject
cd myproject
```

Set up the project structure:

```
mkdir project
mkdir -p src/main/scala
mkdir -p src/main/webapp/WEB-INF
```

Configure sbt:

*project/build.properties:*

```
sbt.version=0.13.5
```

*project/plugins.sbt:*

```scala
addSbtPlugin("com.earldouglas" % "xsbt-web-plugin" % "1.0.0")
```

*build.sbt:*

```scala
name := "myproject"

organization := "myorganization"

version := "0.1.0-SNAPSHOT"

scalaVersion := "2.10.4"

jetty()

libraryDependencies += "javax.servlet" % "javax.servlet-api" % "3.0.1" % "provided"
```

Add a servlet:

*src/main/scala/servlets.scala*:

```scala
package servlets

import scala.xml.NodeSeq
import scala.xml.PrettyPrinter
import javax.servlet.http.HttpServlet
import javax.servlet.http.HttpServletRequest
import javax.servlet.http.HttpServletResponse

class MyServlet extends HttpServlet {

  override def doGet(request: HttpServletRequest, response: HttpServletResponse) {

    response.setContentType("text/html")
    response.setCharacterEncoding("UTF-8")

    val responseBody: NodeSeq =
      <html>
        <body>
          <h1>Hello, world!</h1>
        </body>
      </html>

    val printer = new PrettyPrinter(80, 2)

    response.getWriter.write(printer.formatNodes(responseBody))

  }

}
```

*src/main/webapp/WEB-INF/web.xml*:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/javaee"
	      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	      xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
	      http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
	      version="3.0">

  <servlet>
    <servlet-name>my servlet</servlet-name>
    <servlet-class>servlets.MyServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>my servlet</servlet-name>
    <url-pattern>/*</url-pattern>
  </servlet-mapping>

</web-app>
```

## Tips and tricks

To disable publishing of the *.war* file, add the setting:

```scala
packagedArtifacts <<= packagedArtifacts map { as => as.filter(_._1.`type` != "war") }
```

Note that `package` can still be used to create the *.war* file under the project *target/* directory.

To disable publishing of the project's *.jar* file, add the setting:

```scala
publishArtifact in (Compile, packageBin) := false
```

## Deployment

### Tomcat

On Ubuntu, install both *tomcat7* and *tomcat7-admin*:

```bash
sudo apt-get install tomcat7 tomcat7-admin
```

Create a Tomcat user with the role **manager-script** in */etc/tomcat7/tomcat-users.xml*:

```xml
<user username="manager" password="secret" roles="manager-script" />
```

Restart tomcat:

```bash
sudo service tomcat7 restart
```

Now a WAR file can be deployed using the Manager *deploy* command:

```
curl --upload-file myapp.war "http://manager:secret@myhost:8080/manager/text/deploy?path=/myapp&update=true"
```

The application will be available at *myhost:8080/myapp*.

Learn more about Manager commands [here](http://tomcat.apache.org/tomcat-7.0-doc/manager-howto.html).

### Heroku

1. Install the [Heroku Toolbelt](https://toolbelt.heroku.com/)

2. Install the `heroku-deploy` command line plugin:

```bash
heroku plugins:install https://github.com/heroku/heroku-deploy
```
3. Create a WAR file:

```bash
sbt package
```

4 Deploy the WAR file:

```bash
heroku deploy:war --war <path_to_war_file> --app <app_name>
```

See [devcenter.heroku.com](https://devcenter.heroku.com/articles/war-deployment) for more information.

### Google App Engine

See [developers.google.com](https://developers.google.com/appengine/docs/java/tools/uploadinganapp) for more information.

