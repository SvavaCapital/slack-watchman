<img src="https://i.imgur.com/jeU9F0a.png" width="550">

# Slack Watchman
![Python 2.7 and 3 compatible](https://img.shields.io/pypi/pyversions/slack-watchman)
![PyPI version](https://img.shields.io/pypi/v/slack-watchman.svg)
![License: MIT](https://img.shields.io/pypi/l/slack-watchman.svg)

Monitoring and enumerating Slack for exposed secrets

## About Slack Watchman
Slack Watchman is an application that uses the Slack API to find potentially sensitive data exposed in a Slack workspace, and to enumerate other useful information for red, blue and purple teams.



### Features
Slack Watchman looks for:

- API Keys, Tokens & Service Accounts
  - AWS, Azure, GCP, Google API, Slack (keys & webhooks), Twitter, Facebook, GitHub and more
  - Generic Private keys
  - Access Tokens, Bearer Tokens, Client Secrets, Private Tokens
- Files
    - Certificate files
    - Potentially interesting/malicious/sensitive files (.docm, .xlsm, .zip etc.)
    - Executable files
    - Keychain files
    - Config files for popular services (Terraform, Jenkins, OpenVPN and more)
- Personal Data
    - Leaked passwords
    - Passport numbers, Dates of birth, Social security numbers, National insurance numbers and more
- Financial data
    - Paypal Braintree tokens, Bank card details, IBAN numbers, CUSIP numbers and more

It also enumerates the following:
- User data
    - All users & all admins
- Channel data
    - All channels, including externally shared channels

#### Time based searching
You can run Slack Watchman to look for results going back as far as:
- 24 hours
- 7 days
- 30 days
- All time

This means after one deep scan, you can schedule Slack Watchman to run regularly and only return results from your chosen timeframe.

### Signatures
Slack Watchman uses custom YAML signatures to detect matches in Slack. These signatures are pulled from the central [Watchman Signatures repository](https://github.com/SvavaCapital/watchman-signatures). Slack Watchman automatically updates its signature base at runtime to ensure its using the latest signatures to detect secrets.

### Logging

Slack Watchman gives the following logging options:
- Terminal-friendly Stdout
- JSON to Stdout

Slack Watchman defaults to terminal-friendly stdout logging if no option is given. This is designed to be easier for humans to read.

JSON logging is also available, which is perfect for ingesting into a SIEM or other log analysis platforms.

JSON formatted logging can be easily redirected to a file as below:
```commandline
slack-watchman --timeframe a --all --output json >> slack_watchman_log.json 
```

## Authentication Requirements
### Slack API token
To run Slack Watchman, you will need a Slack API OAuth access token. You can do this by creating a simple [Slack App](https://api.slack.com/apps).

The app needs to have the following **User Token Scopes** added:
```
channels:read
files:read
groups:read
im:read
links:read
mpim:read
remote_files:read
search:read
team:read
users:read
users:read.email
```
**Note**: User tokens act on behalf of the user who authorises them, so I would suggest you create this app and authorise it using a service account, otherwise the app will have access to your private channels and chats.

### Cookie Authentication
Alternatively, Slack Watchman can also authenticate to Slack using a user `d` cookie, which is stored in the browser of each user logged into a workspace.

To use cookie authentication, you will need to provide the `d` cookie, and the URL of the target workspace. Then you will need to use the `--cookie` flag when running Slack Watchman

#### Providing tokens
Slack Watchman will first try to get the Slack token (plus the cookie token and URL if selected) from the environment variables 
- `SLACK_WATCHMAN_TOKEN`
- `SLACK_WATCHMAN_COOKIE`
- `SLACK_WATCHMAN_URL`

If this fails it will try to load the token(s) from `.conf` file (see below).

#### .conf file
Configuration options can be passed in a file named `watchman.conf` which must be stored in your home directory. The file should follow the YAML format, and should look like below:
```yaml
slack_watchman:
  token: xoxp-xxxxxxxx
  cookie: xoxd-%2xxxxx
  url: https://xxxxx.slack.com
```
Slack Watchman will look for this file at runtime, and use the configuration options from here. If you are not using cookie auth, leave `cookie` and `url` blank.

If you are having issues with your .conf file, run it through a YAML linter.

An example file is in `docs/example.conf`

**Note**: Cookie and URL values are optional, and not required if not using cookie authentication.

## Installation
You can install the latest stable version via pip:

```commandline
python3 -m pip install slack-watchman
```

Or build from source yourself:

Download the release source files, then from the top level repository run:
```commandline
python3 -m pip build
python3 -m pip install --force-reinstall dist/*.whl
```

## Docker Image

Slack Watchman is also available from the Docker hub as a Docker image:

`docker pull papermountain/slack-watchman:latest`

You can then run Slack Watchman in a container, making sure you pass the required environment variables:

```commandline
// help
docker run --rm 239700674053.dkr.ecr.ap-south-1.amazonaws.com/stable-saas-v1-common:slack-watchman-latest -h

// scan all
docker run --rm -e SLACK_WATCHMAN_TOKEN=xoxp... 239700674053.dkr.ecr.ap-south-1.amazonaws.com/stable-saas-v1-common:slack-watchman-latest --timeframe a --all --output json
docker run --rm --env-file .env 239700674053.dkr.ecr.ap-south-1.amazonaws.com/stable-saas-v1-common:slack-watchman-latest --timeframe a --all --output stdout
```

## Usage
Slack Watchman will be installed as a global command, use as follows:
```commandline
usage: slack-watchman [-h] --timeframe {d,w,m,a} [--output {json,stdout}] [--version] [--all] [--users] [--channels] [--pii] [--secrets]
                      [--debug] [--verbose] [--cookie]

Monitoring and enumerating Slack for exposed secrets

options:
  -h, --help            show this help message and exit
  --output {json,stdout}, -o {json,stdout}
                        Where to send results
  --version, -v         show program's version number and exit
  --all, -a             Find secrets and PII
  --users, -u           Enumerate users and output them to .csv
  --channels, -c        Enumerate channels and output them to .csv
  --pii, -p             Find personal data: DOB, passport details, drivers licence, ITIN, SSN etc.
  --secrets, -s         Find exposed secrets: credentials, tokens etc.
  --debug, -d           Turn on debug level logging
  --verbose, -V         Turn on more verbose output for JSON logging. This includes more fields, but is larger
  --cookie              Use cookie auth using Slack d cookie. REQUIRES either SLACK_WATCHMAN_COOKIE and SLACK_WATCHMAN_URL environment
                        variables set, or both values set in watchman.conf

required arguments:
  --timeframe {d,w,m,a}, -t {d,w,m,a}
                        How far back to search: d = 24 hours w = 7 days, m = 30 days, a = all time
  ```

You can run Slack Watchman to look for everything, and output to default stdout:

```commandline
slack-watchman --timeframe a --all
```



## License
The source code for this project is released under the [GNU General Public Licence](https://www.gnu.org/licenses/licenses.html#GPL). This project is not associated with Slack Technologies or Salesforce.
