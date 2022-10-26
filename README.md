# DSV Github Actions Demo

This demo uses Delinea DevOps Secrets Vault for retrieval of your secrets.

Now, instead of storing all your secrets directly in your GitHub repo configuration, store the client credentials to connect and retrieve the desired secret or multiple secrets from your secure vault.



## How This Works

## Inputs

| Name           | Description                                              |
| -------------- | -------------------------------------------------------- |
| `domain`       | Tenant domain name (e.g. example.secretsvaultcloud.com). |
| `clientId`     | Client ID for authentication.                            |
| `clientSecret` | Client Secret for authentication.                        |
| `retrieve`     | Data to retrieve from DSV in json format.                |

## Prerequisites

This demo uses authentication based on Client Credentials, i.e. via Client ID and Client Secret.

```shell
rolename="dsv-github-action-demo"
secretpath="ci:dsv-github-action-demo"

dsv role create --name "${rolename}"

clientcred=$(dsv client create --role "${rolename}" --plain | jq -c)

# configure the dsv server, such as mytenant.secretsvaultcloud.com
gh secret set DSV_SERVER

# use the generated client credentials in your repo
gh secret set DSV_CLIENT_ID --body "$( echo "${clientcred}" | jq '.clientId' -r )"
gh secret set DSV_CLIENT_SECRET --body "$( echo "${clientcred}" | jq '.clientSecret' -r )"
```

For further setup, here's how you could extend that script block above with also creating a secret and the policy to read just this secret.

```shell
# Create a secrets
secretkey="dsvdemo"
secretvalue='{"aws_keyid":"YOUR_AWS_ACCESS_KEY_ID","aws_ssecret_access_key":"YOUR_AWS_SECRET_ACCESS_KEY"}'
dsv secret create \
  --path "secrets:${secretpath}:${secretkey}" \
  --data "${secretvalue}" \
  --desc "Secret for AWS Access Key ID and AWS Secret acces key"
  
secretkey="awsbucket"
secretvalue='{"aws_bucket":"YOUR_AWS_BUCKET_NAME"}'
dsv secret create \
  --path "secrets:${secretpath}:${secretkey}" \
  --data "${secretvalue}" \
  --desc "Secret for the AWS S3 bucket name"

# Create a policy to allow role "$rolename" to read secrets
dsv policy create \
  --path "secrets:${secretpath}" \
  --actions 'read' \
  --effect 'allow' \
  --subjects "roles:$rolename" \
  --desc "Policy for the DSV github action demo" \
  --resources "secrets:${secretpath}:<.*>"
```

