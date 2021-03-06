This is a set of templates you can use to set up Continuous Integration for new 
software projects. The intent is not to be a one-size-fits-all solution, but also
not so opinionated that you can't take parts of it and use them in your own system
as needed.

The end result should be that you can use these examples to create an initial
template for your project's CI and then customize them as needed.

---

# Components

Continuous Integration is comprised of multiple components that are combined
together to form one system. The following are some of those components:


## Applications

All CI is intended to build an application and test it. Since different 
applications have different ways of being built and tested, the kind of 
application you have will determine much of the rest of how your CI works.


## Continuous Integration Platforms

A CI Platform is basically a tool or system designed to help you manage
Continuous Integration. The primary feature of these tools is the ability to
run multiple arbitrary 'jobs' to build and test your application. A collection
of these 'jobs' is typically called a 'pipeline'. Once a 'pipeline' finishes
successfully, the Continuous Integration is deemed to have 'passed' successfully.

These CI tools tend to have custom configuration and lots of features designed
to make your life easier. They also require a decent amount of set-up in
combination with all the other components listed herein.

Often there is a way for you to run your builds/tests in a way that is unique 
to a particular CI tool, such as using some custom plugins that do a lot of work
for you. However, beware that the more custom CI functionality you use, the
harder it is to adopt a different tool or work with other components.

If you want to be able to build and test your application on your local machine,
it will be easier if you don't depend heavily on lots of CI-specific functions.
If your local testing deviates from the CI-system testing, you will end up with
code that works locally but not in CI, and vice-versa.


## Source code hosting

CI systems check out source code from a Version Control System (VCS) before
building the source code and then testing it. There are many different VCS
out there, and different hosted providers that will store your code for you.

Managed VCS providers typically have a few different ways a CI system will
interact with them:

### 1. Checking out code
This is the most common interaction between a CI system and VCS. It requires
a *Secret* (see below) to log-in to the VCS and retrieve the code. Often they
support multiple methods to check out code, such as a connection over SSH,
HTTPS, etc, using either usernames/passwords, or SSH keys. Each CI system often
has its own method of setting up and managing this process, and custom features
to check out the code for you (which only saves you a couple of commands).

### 2. Webhooks
When a managed VCS provider receives an update to the source code (such as
from a user pushing code to the server, or opening a pull request), it can
be configured to submit an HTTP Webhook call to a CI system on the public
internet. It may also supply an optional "shared secret" value to authenticate
the webhook call to the CI system.

It's good practice to configure the CI system to only accept such webhook
connections from the IP address range of the VCS provider. However, this is
not actually that secure: any user of the VCS provider can configure a webhook
that hits your CI system, and your system won't know the difference, so always
make sure your CI system validates webhook calls using a secret.

### 3. Build status
CI systems run builds and tests after they have received a webhook update from
a managed VCS provider. At the end of running the build or test, the CI system
has some kind of status of the build job: success, failure, aborted, etc.

Managed VCS providers often have the ability to receive an API call from the
CI system with the build job status. The provider then updates a database
to record the job status of the code that was submitted by the webhook. This
allows users to see the results of CI jobs in the managed provider's interface.

This requires the CI system to be configured with the information of the VCS
provider (repository name, secrets for authentication, etc) and requires the
VCS provider to be set up to allow the updates.


## Secrets

It may be necessary for your CI pipeline to interact with a service using some
sensitive credentials. You should *never* include secrets in your source code
in plain text. For that reason, CI systems need some way to extract a secret
when a CI job runs, and pass it to some task as necessary.

Different kinds of secrets exist, such as:
 - Usernames and Passwords
 - API access tokens
 - SSH keys
 - PGP encryption signing keys

Most CI systems have their own secret storage mechanism. You store your secret
in it by hand and give it a name. Then in your CI jobs, you reference the name
of the secret, and the system will extract the secret value. Usually then it
will provide that secret as an environment variable to the job you're running,
or save it to a file somewhere.

Other times you may be using an external secret storage mechanism, like 
1Password, AWS Secrets Manager, etc. The CI system you use may have a plugin to
interact with that system - but of course, you usually need a secret credential 
just to access that secret storage mechanism... so you would store the credential
for that secret storage machanism in your CI system's secrets storage, and then
retrieve it when your job runs in order to pull more secrets out at run time.

Depending on how you manage your secrets, there may be multiple levels of
set-up needed before you can ever run your CI jobs! (Isn't technology annoying?)


## Build Artifacts

When you build an application and want to test it, it's a good idea to save
that artifact to an artifact repository or package repository. If the testing
succeeds, that copy will be what you'll use to deploy to production. And if
you need to roll-back a bad deployment, the old copy of the build artifact
will be pulled from the artifact/package repository.

A useful aspect of these repositories is that you can often cache components
that you don't want to re-build or download from the internet. Think of it as
your local, safe, secure mirror of software from the internet - as well as
the software you build yourself.

This system usually requires a *Secret* for access (see above).


## Mocks

A 'mock' is a simulated object, service, data, etc. They are also known as fakes
or stubs. Their purpose is to allow you to run tests that would otherwise be
expensive, difficult, unsafe, or unreliable. Mocks are often used in Test-Driven
Development to simulate a result, which code is then written around. Mocks are 
commonly a service like a database or a piece of sample data.

Mocks are sometimes seen as a controversial or unreliable method of testing.
They can be problematic when code is written to work with the mocks, but the
code ends up failing when running against the "real" object the mock is
masquerading as.

Mock services can be set up by a CI system in preparation for an array of tests
and shut down once tests have completed. Data can also be sampled from a live
system at run-time and 'sanitized' to act as a mock.


## Tests

Tests are the main purpose behind Continuous Integration. There are all kinds
of tests you can create for your application. Your tests may be bundled with
your application, or you might use an independent library of tests, or your
application may be tested by some other application's tests.

Tests are a separate from the "build" process of an application. However, you
do need to ensure you're running the tests that match the version of the
built application. Therefore, both the built application and the tests need to
record the same version number so they can verify they're compatible. Otherwise
you could end up with inconsistent testing results.

Tests are typically run after setting up a test environment or mock.


## Code Analysis

As part of building and testing your application, you may want to run some
analysis tools on the code. These tools can do things like check the syntax,
look for common coding problems, possible security holes, accidentally copied
secrets, etc.

These tools are often used as "quality gates" in CI. Each of the "gates" passed
increases the confidence in overall quality of the code. If you want to implement
Continuous Deployment, you want your CI process to result in very high confidence,
so these quality gates can be very important to get right. If a "gate" fails it
should ideally terminate a build/test pipeline.

These code analysis tools may require a *Secret* for access (see above) as well as
some form of configuration for your specific code/project.


---

# Philosophy

There are innumerable ways to accomplish Continuous Integration. Many tools and
systems exist to help with CI, with their own custom-tailored functionality.
This repository follows certain conventions in order to make it easier to combine
different components together easily.


## Don't Repeat Yourself: Maximizing re-use

If you are using one CI tool/system and you want to switch to another, you often
have to re-write a lot of the old system in a format that works for the new system.

Since this repo is dedicated to providing multiple solutions to the same problem,
and we want as much interoperability between components as possible, we need to
avoid re-writing things whenever possible. Toward that end we avoid functionality
that is specific to just one tool or system, and instead use functionality that
can be used in many tools or systems.


### Don't embed shell commands in config files

In many CI systems (Jenkins, CircleCI, TravisCI, GitHub Actions) you can embed
individual shell commands in a configuration file. That makes it easier to keep
everything in one config file, but it makes it harder to then share those same
commands across lots of config files. And since the commands are embedded in
a config file, you can't run a linter on them to ensure they are valid syntax.

In order to avoid that problem, commands are saved into a file that can be run
like a program, and that file is referenced in the CI config file. A Makefile
or shell script (the preferred form) are good places to put those commands.

