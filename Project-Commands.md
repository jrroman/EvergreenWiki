This page is a glossary of Evergreen's supported task commands.

#### shell.exec
This command runs a shell script.

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
* `system_log`: if set to true, the script's output will be written to the task's system logs, instead of inline with logs from the test execution.

#### git.get_project
This command clones the tracked project repository into a given directory, and checks out the revision associated with the task. Also applies patches to the source after cloning it, if the task was created by a patch submission.

NOTE: this command currently only supports clone over SSH.
Make sure your GitHub ssh keys are set up in your target distros.

```yaml
- command: git.get_project
  params:
    directory: src
```

Parameters:
* `dir`: the directory to clone into

#### s3.put
This command uploads a file to Amazon s3, for use in later tasks or distribution.

```yaml
- command: s3.put
  params:
    aws_key: ${aws_key}
    aws_secret: ${aws_secret}
    local_file: src/mongodb-binaries.tgz
    remote_file: mongodb-mongo-master/${build_variant}/${revision}/binaries/mongo-${build_id}.${ext|tgz}
    bucket: mciuploads
    permissions: public-read
    content_type: ${content_type|application/x-gzip}
    display_name: Binaries
```

Parameters:
* `aws_key`: your AWS key (use expansions to keep this a secret)
* `aws_secret`: your AWS secret (use expansions to keep this a secret)
* `local_file`: the local file to post
* `remote_file`: the ec2 path to post the file to
* `bucket`: the ec2 bucket to use
* `permissions`: the ec2 permissions string to upload with
* `content_type`: the MIME type of the file
* `optional`: boolean to indicate if failure to find or upload this file will result in a task failure. Not compatible with local_files_include_filter.
* `display_name`: the display string for the file in the Evergreen UI
* `local_files_include_filter`: used in place of local_file, an array of gitignore file globs. All files that are matched - ones that would be ignored by gitignore - are included in the put.

#### s3.put with multiple files
Using the s3.put command in this uploads multiple files to an s3 bucket. 
```yaml
- command: s3.put
  params:
    aws_key: ${aws_key}
    aws_secret: ${aws_secret}
    local_files_include_filter:
      - slow_tests/coverage/*.tgz
      - fast_tests/coverage/*.tgz
    remote_file: mongodb-mongo-master/test_coverage-
    bucket: mciuploads
    permissions: public-read
    content_type: ${content_type|application/x-gzip}
    display_name: coverage-
```
Each file is displayed in evergreen as the file's name prefixed with the `display_name` field. Each file is uploaded to a path made of the local file's name, in this case whatever matches the `*.tgz`, prefixed with what is set as the `remote_file` field. The filter uses the same specification as gitignore when matching files. In this way, all files that would be marked to be ignored in a gitignore containing the lines `slow_tests/coverate/*.tgz` and `fast_tests/coverate/*.tgz` are uploaded to the s3 bucket.
#### gotest.parse
This command parses Go test results and sends them to the API server.
It accepts files generated by saving the output of the `go test -v` command to a file.

E.g. In a preceding shell.exec command, run  `go test -v > result.suite`

```yaml
- command: gotest.parse_files
  params: 
    files: ["src/*.suite"]
```

Parameters:
* `files`: a list of files (or blobs) to parse and upload

#### attach.results
This command parses results in Evergreen's JSON test result format and posts them to the API server.

The format is as follows:
```json
{
    "results":[
    {
        "status":"pass",
            "test_file":"test_1",
            "exit_code":0,
            "elapsed":0.32200002670288086, //end-start
            "start":1398782500.359, //epoch_time
            "end":1398782500.681 //epoch_time
    },
    {
        "etc":"..."
    },
    ]
}
```

```yaml
- command: attach.results
  params:
    file_location: src/report.json
```

Parameters:
* `file_location`: a .json file to parse and upload


#### attach.xunit_results
This command parses results in the XUnit format and posts them to the API server.

```yaml
- command: attach.xunit_results
  params:
    file: src/results.xml
```

Parameters:
* `file`: a .xml file to parse and upload. A filepath glob can also be supplied to collect results from multiple files.


