# Installation

In its current alpha state, installing evergreen can be a confusing process.
Rest assured that once it is up and running, there will be very little configuration and maintenance for the end user.

##Requirements

#####Install Go From Source
One of Evergreen's most defining features is its reliance on Go's cross compilation support.
In the current release, Go 1.4, cross compilation requires installing Go from source, which you can do by following the instructions [here](https://golang.org/doc/install/source).

####Install MongoDB
Evergreen was built by the developers of MongoDB, so it's no surprise that Evergreen relies on MongoDB for all of its back-end storage needs.
Instructions on install MongoDB are [here](http://docs.mongodb.org/manual/installation/).

####Clone the Repo
Clone the `evergreen-ci/evergreen` repo using git.

We rely on internal vendoring for all of our packages, so there's no need to use `go get` in order to get a copy.

####Build

Compiling the main Evergreen processes is done by simply running `./build.sh`, which will compile them into the `bin` directory.

Cross-compiling the agent is a little trickier. 
Cross-compilation requires building Go from source (see above).
You must then run `go run vendor/src/github.com/laher/goxc/goxc.go -t` to set up your tool chain before any cross compilation can take place.
After this is done, run `./build_agent.sh`, which will compile the agent for all of our supported platforms.


##Configure and Run

Now the fun part.
If any of these directions are confusing or don't seem to work, please [file a GitHub issue](https://github.com/evergreen-ci/evergreen/issues) for this repo;
making Evergreen setup easy and well-documented is the main goal driving our 1.0.0 beta release.

####Config Files

Evergreen relies on a single server-side configuration file for application-wide settings. 
To start, make a copy of the example config file [here](https://github.com/evergreen-ci/evergreen/blob/master/docs/evg_example_config.yml) and fill in the given GitHub tokens and SSH keys.
See our [configuration wiki page](Configuration) for help with setting up your config file.
If you are just trying Evergreen out, feel free to remove the https configuration options to simplify your deployment. 

######Users

Evergreen supports multiple methods of user authentication
 1. Naive Config File Settings
 2. Atlassian Crowd
 3. GitHub OAuth (*Coming Soon*)

If you're just getting stared with Evergreen, we recommend using our naive config-based auth.
Setting up the naive authentication requires adding a user configuration like this
```yaml
auth:
    naive:
        users:
        - username: "myName"
          password: "myPass"
          display_name: "Admin"
        - username: "myName2"
          password: "myPass2"
          display_name: "Rumpelstiltskin"
```
to your config file.
This form of auth is great for getting a small installation up and running, but we do not recommend you use it in production.

Our Atlassian Crowd support works similarly.
Setting up Crowd requires a config resembling
```yaml
auth:
    crowd:
        username: "user"
        password: "PASSPASSPASS"
        urlroot: "https://crowd.domain.com"
```
where the fields are crowd manager credentials.

Finally, for access control, Evergreen has the concept of a "superuser."
Superusers are able to add and modify projects and distros within the Evergreen UI.
Superusers are set by using the field
```yaml
superusers: ["name1", "name2", "name3"]
```
in the root of the config file. 
If the `superusers` field does not exist, all logged in users will have superuser privileges.

#### The Evergreen Processes
Running `build.sh` will generate binaries for `evergreen_ui_server`, `evergreen_api_server`, and `evergreen_runner`. 
Once your config file is in order, you start these processes *from the root of the evergreen git repository* with
```bash
export EVGHOME=`pwd`
bin/evergreen_ui_server -conf path/to/config.yml
bin/evergreen_api_server -conf path/to/config.yml
bin/evergreen_runner -conf path/to/config.yml
```

With these processes running, you can continue setting up your distros and projects through the Evergreen UI.

#####Projects

First, Evergreen needs to know what codebases to test.
Evergreen project configuration is done almost entirely within the repository you are testing.
For more information on writing a project config for your repo, click [here](https://github.com/evergreen-ci/evergreen/wiki/Project-Files).

For the purposes of this tutorial, we will be testing this sample project: https://github.com/evergreen-ci/sample

On the projects page, add a new project, and fill in the required fields to point it to this test repository.
( We recommend project names be simple and not have spaces, as it makes the command line tools easier to use;
save capital letters and spaces for the project's Display Name)

[[images/project_setup.png]]
#######Project Parameters
 * `Enabled/Disabled`: disabling a project stops it from scheduling and running new tasks
 * `Display Name`: how you want the project to be written in the UI (e.g. "My Project")
 * `Config File`: location of project file in the target repository (e.g. "evergreen.yml")
 * `Batch Time`: how many minutes to wait between scheduling new commits (e.g. 120)
 * `Owner`: the GitHub owner of a project (e.g. "mongodb")
 * `Repo Name`: the GitHub repository name (e.g. "mongo-tools")

The variables section let's you define project-level expansions. 
You shouldn't have to add any special variables to run the sample repository.
Click Save Changes.

#####Distros

Next, Evegreen needs to know where to run your tests.
A "distro" in Evergreen is a host configuration that projects can run against.
Distros tell Evergreen how to create and talk to hosts of a certain type.
For example, a user might create a distro called `ubuntu` for spinning up ubuntu instances on ec2,
or she might create a distro `osx-buildfarm` for running tasks on some static OSX machines in her server room.

Evergreen supports both elastic and static computing.
Please check out one or both of the following tutorials before continuing:

For information on configuring a cloud provider along with an Amazon ec2 tutorial, click [here](TODO). (TODO)

For information on configuring a static provider along with a "localhost" tutorial, click [here](https://github.com/evergreen-ci/evergreen/wiki/Static-Tutorial).

With a project and distro set up, make sure your runner and API servers are active, and sit back to watch your tests run.

#####Common pitfalls:

- firewalls and security groups
- knownhosts
- you can check your agent logs by sshing into the working directory
- make sure you have your github keys in order
