# Lets Encrypt certificates via route53

##### This is a simple skeleton for [``dehydrated``](https://github.com/lukas2511/dehydrated/tree/v0.5.0) + [``dehydrated-route53-hook-script``](https://github.com/whereisaaron/dehydrated-route53-hook-script/tree/v0.4.0)

This repo uses these projects directly to:
 1. allow you to use the acme staging server to test your process
 2. allow you to register an acme account key
 3. generate SSL keys & CSRs for each line in ``domains.txt``
 4. fulfill acme ``dns-01`` challenges for each CSR using [``cli53``](https://github.com/barnybug/cli53)
 5. store the keys, CSRs, and signed certs in a configured directory (default ``staged/`` or ``certs/``)

##### There are no scripts in this repository, only configuration files and instructions.

## Setup skeleton

```sh
# full-clone the repository
git clone --recursive https://github.com/SnowblindSystems/dns-01-r53
cd dns-01-r53

# edit  CONTACT_EMAIL  in config.common

# ./d is a symlink to ./dehydrated/dehydrated (inside the git submodule)

# generate & register account key
./d -f config.STAGING --register --accept-terms

# config.PRODUCTION will need its own registration later
```

## AWS Setup
##### Taken almost directly from [whereisaaron/dehydrated-route53-hook-script](https://github.com/whereisaaron/dehydrated-route53-hook-script/tree/v0.4.0#aws-route-53-iam-user-and-policy)

#### Create an IAM policy with the following rights:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "route53:ListResourceRecordSets",
                "route53:ChangeResourceRecordSets"
            ],
            "Resource": "arn:aws:route53:::hostedzone/*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "route53:ListHostedZones",
                "route53:ListHostedZonesByName",
                "route53:ListResourceRecordSets"
            ],
            "Resource": "*"
        },
        {
            "Effect": "Allow",
            "Action": [
                "route53:GetChange"
            ],
            "Resource": "arn:aws:route53:::change/*"
        }
    ]
}
```

##### Optionally restrict the policy to specific zones inside the first Allow:

```json
"Resource": [ 
    "arn:aws:route53:::hostedzone/Z12345678901", 
    "arn:aws:route53:::hostedzone/Z12345678901"
]
```


#### Create an IAM user for programmatic access with the new policy directly attached

#### Add the new user's credentials to ``.credentials``

```sh
export AWS_ACCESS_KEY_ID=AKIA1234567890124567
export AWS_SECRET_ACCESS_KEY=FakeFAKEfakeFAKEfakeFAKEfakeFAKEfakeFAKE
```

#### or ``~/.aws/credentials``

```ini
[default]
aws_access_key_id = AKIA1234567890124567
aws_secret_access_key = FakeFAKEfakeFAKEfakeFAKEfakeFAKEfakeFAKE
```


## Generating + signing certificates

```sh
# edit  domains.txt
# see https://github.com/lukas2511/dehydrated/blob/v0.5.0/docs/domains_txt.md

# install cli53 dependency
brew install cli53 # on macOS
# otherwise see https://github.com/barnybug/cli53

# generate keys + csrs, check cert expiry date, and run challenges
./d -f config.STAGING --cron
```

If that works; repeat using ``config.PRODUCTION``


## Crontab
##### Taken from [whereisaaron/dehydrated-route53-hook-script](https://github.com/whereisaaron/dehydrated-route53-hook-script/tree/v0.4.0#scheduled-task-for-renewals-cron)

The script will only renew certs if they are within [``RENEW_DAYS``](https://github.com/lukas2511/dehydrated/blob/v0.5.0/docs/examples/config#L85) days from expiry.
(default 30).

```cron
@daily path/to/repo/d --config path/to/repo/config.PRODUCTION --cron  >/dev/null
```


## See the main repos for more:
 - https://github.com/lukas2511/dehydrated
 - https://github.com/whereisaaron/dehydrated-route53-hook-script


## Todo
 - Screenshots for AWS setup

