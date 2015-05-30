Project files are how you tell Evergreen what to do.
They contain a set of tasks and variants to run those tasks on, and are stored within the repository they test.
Project files are written in a simple YAML config language.

### Examples
Before reading onward, you should check out some example project files:

1. [Sample tutorial project file](https://github.com/evergreen-ci/sample.git)
2. [Evergreen's own project file](https://github.com/evergreen-ci/evergreen/blob/master/self-tests.yml)
3. [The MongoDB Tools project file](https://github.com/mongodb/mongo-tools/blob/master/common.yml)
4. [The MongoDB Server project file](https://github.com/mongodb/mongo/blob/master/etc/evergreen.yml)

Though some of them are quite large, the pieces that make them up are very simple.

### Basic Features

#### Tasks
A task is any discrete job you want Evergreen to run, typically a build, test suite, or deployment of some kind.
A task is made up of a list of commands/functions.
Currently we include commands for interacting with git, running shell scripts, parsing test results, and manipulating Amazon s3.

For example, a couple tasks might look like:
```yaml
tasks:
- name: compile
  depends_on: []
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

Notice that each task contains:

1. A name
2. A set of dependencies of other tasks
3. A list of commands and/or functions

#### Commands
Commands are the building blocks of tasks.
Each command has a set of parameters that it can take.

#####shell.exec
Example:
```yaml
- command: shell.exec
  params:
    working_dir: src
    script: |
      echo "this is a 2nd command in the function!"
      ls
```

Parameters:
* `script`: the script to run 
* `working_dir`: the directory to execute the shell script in
* `background`: if set to true, does not wait for the script to exit before running the next commands
* `silent`: if set to true, does not log any shell output during execution; useful to avoid leaking sensitive info
* `continue_on_err`: if set to true, causes command to exit with success regardless of the script's exit code

#####shell.track
Example:
```yaml
```
#####shell.cleanup
Example:
```yaml
```
#####git.get_project
Example:
```yaml
```
#####git.apply_patch
Example:
```yaml
```
#####s3.put
Example:
```yaml
```
#####gotest.parse
Example:
```yaml
```
#####attach.results
Example:
```yaml
```
#####attach.x_unit_results
Example:
```yaml
```



#### Expansions

### Advanced Features

#### Expansions
#### Functions
#### The Power of YAML
