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

