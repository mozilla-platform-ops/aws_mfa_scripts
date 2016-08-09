Helper scripts for AWS MFA sessions
===================================

A pair of helper scripts to start and end MFA sessions for using temporary AWS credentials.

`mfs-start-session` uses `aws sts get-session-token` to generate and output temporary credentials
`mfs-end-session` creates (or updates) an inline user policy which prevents use of any existing MFA tokens based on aws:TokenIssueTime

c.f. http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_temp_control-access_disable-perms.html#denying-access-to-credentials-by-issue-time

Requirements
------------
- [awscli](http://docs.aws.amazon.com/cli/latest/userguide/installing.html)
- [jq](https://stedolan.github.io/jq/)
- AWS CLI tools configured (http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration)
- MFA enabled on your user account
- Permission to query the [IAM:ListMFADevices API](http://docs.aws.amazon.com/IAM/latest/APIReference/API_ListMFADevices.html)

An example managed policy that requires MFA for any API call *except* IAM:ListMFADevices:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "RequireMfaUse",
            "Effect": "Deny",
            "NotAction": "IAM:ListMFADevices",
            "Resource": "*",
            "Condition": {
                "Null": {
                    "aws:MultiFactorAuthAge": "true"
                }
            }
        },
        {
            "Sid": "RequireMfaWithinTimeFrame",
            "Effect": "Deny",
            "NotAction": "IAM:ListMFADevices",
            "Resource": "*",
            "Condition": {
                "NumericGreaterThan": {
                    "aws:MultiFactorAuthAge": "28800"
                }
            }
        }
    ]
}
```


Usage
-----
- To start an MFA session:
```bash
$(./mfa-start-session <profile>)
Enter MFA token code for arn:aws:iam::XXXXXXXXXXXX:mfa/USERNAME: XXXXXX
Token expires on 2016-05-03T04:16:24Z
```

- To end a session:
```bash
$(./mfa-ens-session)
Updating inline policy
Clearing session tokens from this shell
```

If you've manually copied and exported the AWS_* environment variables to another shell, do
the following to clear them:
```bash
unset AWS_SESSION_TOKEN AWS_SECRET_ACCESS_KEY AWS_ACCESS_KEY_ID
```

Authors
=======
Created by [Kendall Libby](https://github.com/klibby)

License
=======
Mozilla Public License, version 2.0. See LICENSE for full details.

