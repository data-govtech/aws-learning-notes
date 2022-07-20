# Setup IAM for Programmatic Access with 2FA



## Create Developer User in AWS Console



1. Create a role `u-role-prog-access-developer`

   * Set `Trust Entities` to current account
   * Enable 2FA: set `aws:MultiFactorAuthPresent` to `true` 

   ![image-20220110174243124](https://raw.githubusercontent.com/qinjie/picgo-images/main/image-20220110174243124.png)

   * Add minimal access rights

   ![image-20220421150600740](https://raw.githubusercontent.com/qinjie/picgo-images/main/image-20220421150600740.png)

2. Create a policy `u-iam-policy-prog-access-developer` to resume above role `u-role-prog-access-developer`.

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": "sts:AssumeRole",
               "Resource": "arn:aws:iam::XXXXXXXXX:role/u-role-prog-access-developer",
               "Condition": {
                   "BoolIfExists": {
                       "aws:MultiFactorAuthPresent": "true"
                   }
               }
           }
       ]
   }
   ```

3. Create a user group `developers` and attach above policy `u-iam-policy-prog-access-developer` to this user group.

4. Create user, e.g. `qinjie-capdev`, and add user to above group.



## Configure CLI Profile

1. Setup profile a profile `capdev` (or any other name) using `aws configure`.

2. Edit `~/.aws/credential` file by 

   ```
   [capdev]
   aws_access_key_id = <ACCESS KEY_I
   aws_secret_access_key = <SECRET_ACCESS_KEY>
   source_profile = capdev
   mfa_serial = arn:aws:iam::<AWS_ACCOUNT_ID>:mfa/qinjie-capdev
   role_arn = arn:aws:iam::<AWS_ACCOUNT_ID>:role/u-role-prog-access-developer
   ```

3. In `~/.aws/config`, it should be:

   ```
   [profile capdev]
   region = ap-southeast-1
   output = json
   ```

   

