Errors and Fixes when running docker agent
==========================================
ERROR1: When we add docker agent to the pipeline we might get the following error
------
org.codehaus.groovy.control.MultipleCompilationErrorsException: startup failed:
WorkflowScript: 3: Invalid agent type "docker" specified. Must be one of [any, label, none] @ line 3, column 9.
           docker {
           ^

1 error

	at org.codehaus.groovy.control.ErrorCollector.failIfErrors(ErrorCollector.java:309)
	at org.codehaus.groovy.control.CompilationUnit.applyToPrimaryClassNodes(CompilationUnit.java:1107)
	at org.codehaus.groovy.control.CompilationUnit.doPhaseOperation(CompilationUnit.java:624)
	at org.codehaus.groovy.control.CompilationUnit.processPhaseOperations(CompilationUnit.java:602)
	at org.codehaus.groovy.control.CompilationUnit.compile(CompilationUnit.java:579)
	at groovy.lang.GroovyClassLoader.doParseClass(GroovyClassLoader.java:323)
	at groovy.lang.GroovyClassLoader.parseClass(GroovyClassLoader.java:293)
	at PluginClassLoader for script-security//org.jenkinsci.plugins.scriptsecurity.sandbox.groovy.GroovySandbox$Scope.parse(GroovySandbox.java:163)
	at PluginClassLoader for workflow-cps//org.jenkinsci.plugins.workflow.cps.CpsGroovyShell.doParse(CpsGroovyShell.java:190)
	at PluginClassLoader for workflow-cps//org.jenkinsci.plugins.workflow.cps.CpsGroovyShell.reparse(CpsGroovyShell.java:175)
	at PluginClassLoader for workflow-cps//org.jenkinsci.plugins.workflow.cps.CpsFlowExecution.parseScript(CpsFlowExecution.java:652)
	at PluginClassLoader for workflow-cps//org.jenkinsci.plugins.workflow.cps.CpsFlowExecution.start(CpsFlowExecution.java:598)
	at PluginClassLoader for workflow-job//org.jenkinsci.plugins.workflow.job.WorkflowRun.run(WorkflowRun.java:335)
	at hudson.model.ResourceController.execute(ResourceController.java:101)
	at hudson.model.Executor.run(Executor.java:446)
[withMaven] downstreamPipelineTriggerRunListener - Failure to introspect build steps: java.io.IOException: docker-agent-pipeline #1 did not yet start

SOLUTION:
1. Add the missing plugins if not already installed
   a. Docker pipleine
   b. Docker Plugin
2. Restart the jenkins

ERROR2: The pipeline recognizes the docker keyword. But it fails to find the docker runtime.
------

For Jenkins installed using HomeBrew sometimes we get error when trying to run a docker jenkins agent.

The error message would say something like
=============================================
+ docker inspect -f . maven:latest
/Users/sankalp/.jenkins/workspace/docker-agent-pipeline@tmp/durable-5d1a8e7b/script.sh.copy: line 1: docker: command not found

SOLUTION:
1. Step1: Find the location of jenkins 
   brew --prefix jenkins-lts
2. Step 2: Find the location of he file ‘homebrew.mxcl.jenkins-lts.plist’ file situated at the path -> /opt/homebrew/ opt/jenkins-lts (Output of step 1)
3. Step 3: Edit the file to include the location of docker executable on the local mac
   References: https://medium.com/@bectorhimanshu/fix-for-the-error-docker-command-not-found-when-building-docker-image-using-jenkins-pipeline-on-db1bc5fa7976
   https://stackoverflow.com/questions/40043004/docker-command-not-found-mac-mini-only-happens-in-jenkins-shell-step-but-wo/58688536#58688536

   The file after editing will look like this
   <?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
  <dict>
    <key>Label</key>
    <string>homebrew.mxcl.jenkins-lts</string>
    <key>ProgramArguments</key>
    <array>
      <string>/usr/libexec/java_home</string>
      <string>-v</string>
      <string>1.8</string>
      <string>--exec</string>
      <string>java</string>
      <string>-Dmail.smtp.starttls.enable=true</string>
      <string>-jar</string>
      <string>/usr/local/opt/jenkins-lts/libexec/jenkins.war</string>
      <string>--httpListenAddress=127.0.0.1</string>
      <string>--httpPort=8080</string>
    </array>
    <key>RunAtLoad</key>
    <true />
    <!-- Start of New content Addition-->
    <key>EnvironmentVariables</key>
    <dict>
      <key>PATH</key>
      <string>/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:/Applications/Docker.app/Contents/Resources/bin/:/Users/Kh0a/Library/Group\ Containers/group.com.docker/Applications/Docker.app/Contents/Resources/bin</string>
    </dict>
    <!-- End of New content Addition-->
  </dict>
</plist>

ERROR3: The pipeline recognizes the docker runtime but still we get an error like below
------
+ docker inspect -f . maven:latest

Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?
[Pipeline] isUnix

SOLUTION:
The above error occurs when the docker daemon is not running. Restart the docker and the error will go away and the build will succeed


