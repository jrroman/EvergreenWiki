## Config Files
Evergreen relies on a shared config file for the API, UI, and Runner processes.
This config file tells Evergreen how to talk to GitHub, how to authenticate users, how to interact with external hosts and host providers, how to send email notifications, and so on.
You can find a sample config with spaces for you to fill in [here](https://github.com/evergreen-ci/evergreen/blob/master/docs/evg_example_config.yml).

### Fields
##### Database Connection
These fields control which MongoDB instance Evergreen talks to.
```yaml
dburl: "mongodb://localhost:27017"
db: "evg"
```

`dburl`: the MongoDB connection string to use.

`db`: the MongoDB database to use.

##### Cloud Providers
######AWS
Amazon services require a an ID and a Secret, which are set as follows:
```yaml
providers:
    aws:
        aws_id: PASTE_AWS_ID
        aws_secret: PASTE_AWS_SECRET
```

##### Auth

##### Github
Set your GitHub token with
```yaml
credentials:
    github: PASTE_TOKEN
```

##### SSH Keys
You can have several different private keys for communicating with different cloud and static host providers.
They are set with:
```yaml
keys:
    main: "/path/to/your/key.pem"
    whatever: "/path/to/another.pem"
```

##### Runner Process
The Runner process runs all of the main work within Evergreen, from polling github to starting tests.
It has one main configurable variable: the number of seconds to wait before starting its process loop.
```yaml
runner:
    intervalseconds: 120
```
Lowering this number will make evergreen run more frequently, with the risk of more lock contention from the API server.

##### API Settings
There is a slight quirk to API configuration in `0.9.0`.
In your settings file, there is a root options called `motu`. 
MOTU stands for "master of the universe", and is a holdover from the early days of Evergreen.
In the config, `motu` is the address of the API server.
It is the address that agents communicate with once they are running on their respective hosts.
It could look like
```yaml
motu: "http://localhost:8080"
```
for a totally local Evergreen deployment, or something more like
```yaml
motu: "http://evergreen.mycompany.com:8080"
```

The rest of the API config is handled in the `api` field. 
```yaml
    logfile: "path/to/api.log"
    httplistenaddr: ":8080"
    httpslistenaddr: ":8443"
    httpskey: |
        -----BEGIN RSA PRIVATE KEY-----
        paste a key here if you want https communication between agents and the API server
        -----END RSA PRIVATE KEY-----
```

##### UI Settings
These settings control the Evergreen web interface.
```yaml
ui:
    logfile: "path/to/ui.log"
    secret: "here is my secret"
    cachetemplates: true
    defaultproject: "mongodb-mongo-master"
    url: "http://evergreen.mycompany.com"
    helpurl: "https://urlforhelp"
    httplistenaddr: ":9090"
```

`logfile`: file to write UI logs to

`cachetemplates`: boolean, determines whether templates are re-parsed on every page load (turn this off for development work)

`secret`: secret string for your cookies

`defaultproject`: the default project to load up when a user first loads the main UI page

`url`: the absolute URL of the UI server

`helpurl`: use this to overwrite the help link, to point it somewhere other than the GitHub wiki

`httplistenaddr`: the address for the UI server to listen on (in form of `:number`)


##### Notification / E-mail
These settings tell Evergreen how to send mail. 
You deployment will run if these are unset, but you won't be notified of system errors if email is not set up.
```yaml
notify:
    smtp:
        from: "name@yourdomain.com"
        server: "localhost"
        port: 25
        use_ssl: false
        admin_email:
        - test@test.test
```
`smtp.from`: the address to send e-mail from

`smtp.server`: the address of your SMTP server

`smtp.port`: the port of your SMTP server

`smtp.use_ssl`: boolean, tells the SMTP server to communicate with SSL

`smtp.admin_email`: a list of addresses to send notifications to if Evergreen processes crash or error in some way


##### Expansions
You can use this section to specify parameters for use in distro setup scripts or tasks. 
```yaml
expansions: 
    github_private_key: |-
        -----BEGIN RSA PRIVATE KEY-----
        Your private key to copy over to machines you spin up,
        so that they can talk to GitHub with ssh.
        -----END RSA PRIVATE KEY-----
```
Expansions are defined in simple `key: value` yaml syntax.
