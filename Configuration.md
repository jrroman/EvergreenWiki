## Config Files
Evergreen relies on a shared config file for the API, UI, and Runner processes.
This config file tells Evergreen how to talk to GitHub, how to authenticate users, how to interact with external hosts and host providers, how to send email notifications, and so on.
You can find a sample config with spaces for you to fill in [here](https://github.com/evergreen-ci/evergreen/blob/master/docs/evg_example_config.yml).

### Fields
##### Database Connection
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


