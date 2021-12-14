To use Jenkins as your CI platform, you typically need the following set up.

 - The Jenkins software. You can download this from the web, or get it packaged
   in a Linux distribution or container image.

 - Something to run the Jenkins software. You can use a plain-old virtual
   machine, or something that runs containers for you (Docker, AWS ECS, Hashicorp
   Nomad, Kubernetes, etc). 

 - Jenkins plugins installed to use your VCS of choice or VCS managed provider
   (Git, GitHub, Bitbucket, Mercurial, etc). Usually you want the plugin for a
   VCS managed provider specifically, as that will allow Jenkins to update the
   build status on the VCS provider.

 - Secrets allowing you to pull code from the VCS (either an SSH key that has
   access to the code, or a username/password that has access). These secrets
   need to be configured in Jenkins's credential store, or accessed via a
   Jenkins plugin that retrieves secrets from somewhere (ex: AWS Secrets Manager
   Credential Manager plugin for Jenkins).

 - Configuration for Jenkins to pull source code and configure jobs to run.
   This is a bit of a chicken-and-egg problem; you want to run your Jenkins
   jobs, but your Jenkins jobs (and configuration) are in version control, but
   you need to configure Jenkins to use your version control!
   
   The most common way to do this is to install the Jenkins Configuration as
   Code plugin, configure a JCasC file with the basic Jenkins server configuration,
   configure a JobDSL job to pull Jenkinsfile jobs periodically from your VCS,
   and configure the Jenkins server with some SSH credentials to be able to pull
   the configuration and jobs from the VCS server.

Optional set-up steps:

 - A webhook set up on the VCS managed provider that will update Jenkins when
   code has been updated.

 - Incoming network access so that the VCS managed provider can send webhooks
   to Jenkins.
