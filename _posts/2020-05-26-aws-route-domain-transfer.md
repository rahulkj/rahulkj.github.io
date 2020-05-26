---
layout: post
title:  "Transfer AWS domain from one account to another using AWS CLI"
date:   2020-05-26 11:20:00 -0600
categories: technology
---

Procedure to transfer the AWS domain from one AWS account to another AWS account

## Prerequisites
- Install AWS cli but following AWS documentation

## Initiate transfer from old account to new account

To initiate the transfer of a domain, gather the following information:
- Domain name _(domain you want to transfer, exaple test.com)_
- Accout Id _(AWS account id of the target AWS account)_

Once you have the information handy, you can execute the command:
- `aws configure` _(configure with your source/old AWS account details)_
- `aws route53domains transfer-domain-to-another-aws-account --domain-name test.com --account-id 123456789P`

Once the request is successful, you will get the output as below:
```
{
    "OperationId": "062c2083-ea11-1230-b7c6-5df89e0b2486",
    "Password": "K0xwW`}M+[br''"
}
```

The above output means, your request for transfer the domain was initated and is waiting on the next step, where the target AWS account, needs to accept the transfer.

## Accept the domain transfer from the target account

To accept the domain being transferred from the old/source AWS account, you need to first log into the new/target AWS account.
- `aws configure` _(configure with your new/target AWS account details)_
- `aws route53domains accept-domain-transfer-from-another-aws-account --cli-input-json "$(cat accept-domain.json)"`

where the contents of `accept-domain.json` are:
```
{
    "DomainName": "test.com",
    "Password": "K0xwW`}M+[br''"
}
```

Paste the password from the previous command into this json, and run the command. Once successful, ensure you get a response with a `OperationId`

## Cancel domain transfer

You might have a need to cancel the domain transfer, and its not that hard.

- `aws route53domains cancel-domain-transfer-to-another-aws-account --domain-name "test.com"`

## Troubleshooting
- If you received `The security token included in the request is invalid.`, most likely your aws credentials have expired. So correct them and try again.