Project files are how you tell Evergreen what to do.
They contain a set of tasks and variants to run those tasks on, and are stored within the repository they test.
Project files are written in a simple YAML config language.

## Examples
Before reading onward, you should check out some example project files:

1. [Sample tutorial project file](https://github.com/evergreen-ci/sample.git)
2. [Evergreen's own project file](https://github.com/evergreen-ci/evergreen/blob/master/self-tests.yml)
3. [The MongoDB Tools project file](https://github.com/mongodb/mongo-tools/blob/master/common.yml)
4. [The MongoDB Server project file](https://github.com/mongodb/mongo/blob/master/etc/evergreen.yml)

Though some of them are quite large, the pieces that make them up are very simple.

## Basic Features

### Tasks
A task is any discrete job you want Evergreen to run, typically a build, test suite, or deployment of some kind.
They are the smallest unit of parallelization within Evergreen.
Each task is made up of a list of commands/functions.
Currently we include commands for interacting with git, running shell scripts, parsing test results, and manipulating Amazon s3.

For example, a couple of tasks might look like:
```yaml
tasks:
- name: compile
  commands:
    - command: git.get_project
      params:
        directory: src
    - func: "compile and upload to s3" 
- name: passing_test 
  depends_on: 
  - name: compile
  commands:
    - func: "download compiled artifacts"
    - func: "run a task that passes"
```

Notice that tasks contain:

1. A name
2. A set of dependencies of other tasks
3. A list of commands and/or functions that tell Evergreen how to run it

#### Commands
Commands are the building blocks of tasks.
They do things like clone a project, download artifacts, and execute arbitrary shell scripts.
Each command has a set of parameters that it can take.
A full list of commands and their parameters is accessible [here](Project-Commands).

#### Functions
Functions are a simple way to group a set of commands together for reuse.
They are defined within the file as
```yaml

functions:
  "function name":
    - command: "command.name"
    - command: "command.name2"
    # ...and so on


  # a real example from Evergreen's tests:
  "start mongod":
      - command: shell.exec
        params:
          background: true
          script: |
            set -o verbose
            cd mongodb
            echo "starting mongod..."
            ./mongod${extension} --dbpath ./db_files &
            echo "waiting for mongod to start up"
      - command: shell.exec
        params:
          script: |
            cd mongodb 
            ./mongo${extension} --nodb --eval 'assert.soon(function(x){try{var d = new Mongo("localhost:27017"); return true}catch(e){return false}}, "timed out connecting")'
            echo "mongod is up."
```

and they are referenced within a task definition by

```yaml
- name: taskName
  commands:
  - func: "function name"

  # real example from the MongoDB server
  - func: "run tests"
    vars:
      resmoke_args: --help
      run_multiple_jobs: false
```

Notice that the function reference can define a set of `vars` which are treated as expansions within the configuration of the commands in the function.


### Build Variants
Build variants are a set of tasks run on a given platform. 
Each build variant has control over which tasks it runs, what distro it runs on, and what expansions it uses.

```yaml
buildvariants:
- name: osx-108
  display_name: OSX
  run_on:
  - localtestdistro
  expansions:
    test_flags: "blah blah"
  tasks:
  - name: compile
  - name: passing_test
  - name: failing_test
  - name: timeout_test
- name: ubuntu
  display_name: Ubuntu
  run_on:
  - ubuntu1404-test
  expansions:
    test_flags: "blah blah"
  tasks:
  - name: compile
  - name: passing_test
  - name: failing_test
  - name: timeout_test
```
Fields:
* `name`: an identification string for the variant
* `display_name`: how the variant is displayed in the Evergreen UI
* `run_on`: a list of acceptable distros to run the variant on (usually of length 1)
* `expansions`: a set of key-value expansion pairs
* `tasks`: a list of tasks to run

Additionally, an item in the `tasks` list can be of the form
```yaml
tasks:
- name: compile
- run_on: ubuntu1404-build
```
allowing tasks within a build variant to be run on different distros.
This is useful for optimizing tasks like compilations, that can benefit from larger, more powerful machines.

## Advanced Features

These features will help you do more complicated workloads with Evergreen.

### Pre, Post, and Timeout

All projects can have a `pre` and `post` field which define a list of command to run at the start and end of every task.
These are incredibly useful as a place for results commands or for `shell.track`/`shell.cleanup`.

```yaml
pre:
  - command: shell.track

post:
  - command: attach.results
    params:
      file_location: src/report.json
  - command: shell.cleanup
```

Additionally, project configs offer a hook for running command when a task times out,
allowing you to automatically run a debug script when something is stuck.

```yaml
timeout:
  - command: shell.exec
    params:
      working_dir: src
      script: |
        echo "Calling the hang analyzer..."
        python buildscripts/hang_analyzer.py
```

You can customize the points at which the "timeout" conditions are triggered.
To cause a task to stop (and fail) if it doesn't complete within an allotted time, set the key `exec_timeout_secs` on the project file to the maximum allowed length of execution time. This timeout defaults to *6 hours*.

You may also force a specific command to trigger a failure if it does not appear to generate any output on `stdout`/`stderr` for more than a certain threshold, using the `timeout_secs` setting on the command. As long as the command does not appear to be idle it will be allowed to continue, but if it does not write any output for longer than `timeout_secs` then the timeout handler will be triggered.

Example:

```yaml
exec_timeout_secs: 60 # automatically fail any task if it takes longer than a minute to finish.
buildvariants:
- name: osx-108
  display_name: OSX
  run_on:
  - localtestdistro
  tasks:
  - name: compile

tasks:
  name: compile
  commands:
    - command: shell.exec
      timeout_secs: 10 # force this command to fail if it stays "idle" for 10 seconds or more
      params:
        script: |
          sleep 1000
```


exec_timeout_secs

### Expansions

Expansions are variables within your config file. 
They take the form `${key_name}` within your project,
and are defined on a project-wide level on the project configuration page or on a build variant level within in the project.
They can be used an inputs to commands, including shell scripts.
```yaml
command: s3.get
   params:
     aws_key: ${aws_key}
     aws_secret: ${aws_secret}
```

Expansions can also take default arguments, in the form of `${key_name|default}`.
```yaml
 command: shell.exec
    params:
      working_dir: src
      script: |
        if [ ${has_pyyaml_installed|false} = false ]; then 
        ...
```
If an expansion is used in your project file, but is unset, it will be replaced with its default value.
If there is no default value, the empty string will be used.

##### Default Expansions
Every task has some expansions available by default:

* `${is_patch}` is "true" if the running task is a patch build, "" if it is not
* `${task_id}` is the task's unique id
* `${task_name}` is the name of the task
* `${execution}` is the execution number of the task (how many times is has been reset)
* `${build_id}` is the id of the build the task belongs to
* `${build_variant}` is the name of the build the task belongs to
* `${version_id}` is the id of the task's version
* `${workdir}` is the distro's working directory
* `${revision}` is the git sha for the tested revision
* `${project}` is the is for the project the task belongs to
* `${branch_name}` is the name of the branch being tested by the project


### Task Tags
Most projects have some implicit grouping at every layer.
Some tests are integration tests, others unit tests; features can be related even if their tests are stored in different places.
Evergreen provides an interface for manipulating tasks using this kind of reasoning through *tag selectors.*

Tags are defined as an array as part of a task definition.
Tags should be self-explanatory and human-readable.
```yaml
tasks:
  # this task is an integration test of backend systems; it requires a running database
- name: db
  tags: ["integration", "backend", "db_required"]
  commands:
    - func: "do test"

  # this task is an integration test of frontend systems using javascript
- name: web_admin_page
  tags: ["integration", "frontend", "js"]
  commands:
    - func: "do test"

  # this task is an integration test of frontend systems using javascript
- name: web_user_settings
  tags: ["integration", "frontend", js]
  commands:
    - func: "do test"
```

Tags can be referenced in variant definitions to quickly include groups of tasks.
```yaml
buildvariants:
  # this project only does browser tests on OSX
- name: osx
    display_name: OSX
    run_on:
    - osx-distro
    tasks:
    - ".frontend"
    - ".js"

  # this variant does everything
- name: ubuntu
    display_name: Ubuntu
    run_on:
    - ubuntu-1440
    tasks:
    - "*"

  # this experimental variant runs on a tiny computer and can't use a database or run browser tests
- name: ubuntu_pi
    display_name: Ubuntu Raspberry Pi
    run_on:
    - ubuntu-1440
    tasks:
    - "!.db_required !.frontend"
```

Tags can also be referenced in dependency definitions.
```yaml
tasks:
  # this project only does long-running performance tests on builds with passing unit tests
- name: performance
  depends_on:
  - ".unit"
  commands:
    - func: "do test"

  # this task runs once performance and integration tests finish, regardless of the result
- name: publish_binaries
  depends_on:
  - name: performance
    status: *
  - name: ".integration"
    status: *
```

Tag selectors are used to define complex select groups of tasks based on user-defined tags.
Selection syntax is currently defined as a whitespace-delimited set of criteria, where each criterion is a different name or tag with optional modifiers.
Formally, we define the syntax as:
```
Selector := [whitespace-delimited list of Criterion]
Criterion :=  (optional ! rune)(optional . rune)<Name> or "*" // where "!" specifies a negation of the criteria and "." specifies a tag as opposed to a name
Name := <any string> // excluding whitespace, '.', and '!'
```

Selectors return all items that satisfy all of the criteria.
That is, they return the *set intersection* of each individual criterion.

For example:
* `red` would return the item named "red"
* `.primary` would return all items with the tag "primary"
* `!.primary` would return all items that are NOT tagged "primary"
* `.cool !blue` would return all items that are tagged "cool" and NOT named "blue"
* `.cool !.primary` would return all items that are tagged "cool" and NOT tagged "primary"
* `*` would return all items

### Complex Dependencies / Requires

Some projects need access to complicated dependency structures in order to function.
Consider a build with tasks A, B, C, D, and E.
Task A creates an external resource; Tasks B, C, and D use that resource; Task E destroys the resource.
For normal builds, this works fine, but allowing patches against a configuration like this can prove dangerous,
as dependencies cannot guarantee that E will be scheduled if A is.

Evergreen task configurations have a `requires` field for cases like this.
The `requires` field allows tasks to require other tasks in to be included when creating patches.
Additionally, dependencies can specify a `patch_optional` flag to denote that a dependency of a task does not
have to be created automatically when a patch is configured, so that required tasks do not have to schedule all
of their dependencies.
Using these two features, we can build a config for the above dependency structure that allows a user to
schedule task B and have A and E automatically added.

```yaml
tasks:
- name: A
  requires:
  - name: E

- name: B
  depends_on:
  - name: A

- name: C
  depends_on:
  - name: A

- name: D
  depends_on:
  - name: A

- name: E
  depends_on:
  - name: A
  - name: B
    patch_optional: true
  - name: C
    patch_optional: true
  - name: D
    patch_optional: true
```

### Ignoring Changes to Certain Files
Some commits to your repository don't need to be tested.
The obvious examples here would be documentation or configuration files for other Evergreen projects—changes to README.md don't need to trigger your builds.
To address this, project files can define a top-level `ignore` list of gitignore-style globs which tell Evergreen to not automatically run tasks for commits that only change ignored files.
```yaml
ignore:
    - "version.json" # don't schedule tests for changes to this specific file
    - "*.md" # don't schedule tests for changes to any markdown files
    - "*.txt" # don't schedule tests for changes to any txt files
    - "!testdata/sample.txt" # EXCEPT for changes to this txt file that's part of a test suite
```
In the above example, a commit that only changes `README.md` would not be automatically scheduled, since `*.md` is ignored. A commit that changes both `README.md` and `important_file.cpp` _would_ schedule tasks, since only some of the commit's changed files are ignored.

Full gitignore syntax is explained [here](https://git-scm.com/docs/gitignore). Ignored versions may still be scheduled manually, and their tasks will still be scheduled on failure stepback.

### The Power of YAML
YAML as a format has some built-in support for defining variables and using them.
You might notice the use of node anchors and references in some of our project code.
For a quick example, see: http://en.wikipedia.org/wiki/YAML#Reference 

