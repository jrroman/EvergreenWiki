How to set up and use the command-line tool
==

Downloading the command line tool
--

Go to your evergreen user settings page (accessible the drop-down in the upper right, or at `/settings`) to find links to download the binaries, if the server admin has made them available.
Copy and paste the text in the configuration panel on the settings page into a file in your *home directory* called `.evergreen.yml`, which will contain the authentication information needed for the client to access the server.

Basic Usage
--

To submit a patch, run this from your local copy of the mongodb/mongo repo:
```bash
evergreen patch -p <project-id>
```
    
Variants and tasks for a patch can be specified with the `-v` and `-t`:
```bash
evergreen patch -v enterprise-suse11-64 -t compile
```

Multiple tasks and variants are specified by passing the flags multiple times:
```bash
evergreen patch -v enterprise-suse11-64 -v solaris-64-bit -t compile -t unittest -t jsCore
```

And _every_ task or variant can be specified by using the "all" keyword:
```bash
evergreen patch -v all -t all
```

*NOTE:* The first time you run a patch, you'll be asked if you want to set the given parameters as your default project and variants.
After setting defaults, you can omit the flags and the default values will be used, so that just running `evergreen patch` will work.

Defaults may be changed at any time by editing your `~/.evergreen.yml` file.


Advanced Patch Tips
--

#####Multiple Defaults
While the `evergreen` program has no built-in method of saving multiple configurations of defaults for a project, you can easily mimic this functionality by using multiple local evergreen configurations.
The command line tool allows you to pass in a specific config file with `--config`:
```bash
evergreen --config ~/.many_compiles.yml patch
```
You can use this feature along with shell aliasing to easily manage multiple default sets.

For example, an enterprising server engineer might create a config file called `tests.yml` with the content
```yaml
api_server_host: #api
ui_server_host: #ui
api_key: #apikey
user: #user
projects:
- name: mongodb-mongo-master
  variants:
  - windows-64-2k8-debug
  - enterprise-rhel-62-64-bit
  tasks:
  - all
```
so that running `evergreen --config tests.yml patch` defaults to running all tasks for those two variants.

You might also want to create a config called `compile.yml` with
```yaml
api_server_host: #api
ui_server_host: #ui
api_key: #apikey
user: #user
projects:
- name: mongodb-mongo-master
  variants:
  - windows-64-2k8-debug
  - enterprise-rhel-62-64-bit
  - solaris-64-bit
  - enterprise-osx-108 #and so on...
  tasks:
  - compile
  - unittests
```
for running basic compile/unit tasks for a variety of platforms with `evergreen --config compile.yml patch`.
This setup also makes it easy to do scripting for nice, automatic patch generation.

#####Git Diff
Extra args to the `git diff` command used to generate your patch may be specified by appending them after `--`.  For example, to generate a patch relative to the previous commit:

      evergreen patch -- HEAD~1

Or to patch relative to a specific tag:

      evergreen patch -- r3.0.2

Though keep in mind that the merge base must still exist in the canonical GitHub repository so that Evergreen can apply the patch.


The `--` feature can also be used to pass flags to `git diff`.
One common use of this is to pass the `--binary` flag to enable patches with binary files, which can come up when checking in external libraries or tests with .bson data.

      evergreen patch -- --binary --full-index


Operating on existing patches
--

To list patches you've created:

      `evergreen list-patches`


To cancel a patch:
 
	  `evergreen cancel-patch -i <patch_id>`
    
To finalize a patch:
 
      `evergreen finalize-patch -i <patch_id>`


To add changes to a module on top of an existing  patch:

     ```
      cd ~/projects/module-project-directory
      evergreen set-module -i <patch_id> -m <module-name>
      ```

### Server Side (for Evergreen admins)

To enable auto-updating of client binaries, add a section like this to the settings file for your server:


```yaml
api:
    clients:
        latest_revision: "c0110ba937047f99c9a68470f6ec51fc6d98c5cc"
        client_binaries:
           - os: "darwin"
             arch: "amd64"
             url: "https://.../evergreen"
           - os: "linux"
             arch: "amd64"
             url: "https://.../evergreen"
           - os: "windows"
             arch: "amd64"
             url: "https://.../evergreen.exe"
```

The "url" keys in each list item should contain the appropriate URL to the binary for each architecture. The "latest_revision" key should contain the githash that was used to build the binary. It should match the output of `evergreen version` for *all* the binaries at the URLs listed in order for auto-updates to be successful.
