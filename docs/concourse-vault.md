Using Vault as Credential Manager with Concourse
---

- Deploy Vault release using the [vault.yml](https://github.com/rahul-kj/concourse-vault/blob/master/vault.yml)
- Create a mount in value for use by concourse pipelines
  `vault mount -path=$CONCOURSE_VAULT_MOUNT -description="Secrets for use by concourse pipelines" generic`

  CONCOURSE_VAULT_MOUNT default value is /concourse, and you can specify your own mount here

- Create a policy file with the following contents > policy
```
path "concourse/*" {
  policy = "read"
  capabilities =  ["read", "list"]
}
```

- Register the policy with vault `vault policy-write policy-name vault-policy.hcl`
- If - Using Periodic Token for Authentication
  - Initialize vault: `vault init`
  - Create a periodic token
    `vault token-create --policy=policy-name -period="600h" -format=json`
  - copy the token value from above and set it in the concourse deployment manifest

    ```
    instance_groups:
    - name: web ...
      jobs:
      - name: atc
        release: concourse
        properties: ...
          vault:
            path_prefix: ((CONCOURSE_VAULT_MOUNT))
            url: ((VAULT_ADDR))
            auth:
              client_token: ((CLIENT_TOKEN))
    ```

- If - Using appRole for Authentication
  - Enable `approle`: `vault auth-enable approle`
  - Export the name of the role that you would like to use
    - `export ROLE_NAME=concourse-role`
  - Create a role and fetch the role-id
    - `vault read -format=json auth/approle/role/$ROLE_NAME/role-id`
  - Fetch the secret-id for the role created above
    - `vault write -format=json -f auth/approle/role/$ROLE_NAME/secret-id`
  - copy the `role-id` and the `secret-id` values from above and set it in the concourse deployment manifest
  - `BACKEND_ROLE` will be `approle` in this case
    ```
    instance_groups:
    - name: web ...
      jobs:
      - name: atc
        release: concourse
        properties: ...
          vault:
            path_prefix: ((CONCOURSE_VAULT_MOUNT))
            url: ((VAULT_ADDR))
            auth:
              backend: ((BACKEND_ROLE))
              params:
                role_id: ((ROLE_ID))
                secret_id: ((SECRET_ID))
    ```

- deploy concourse
- populate all the variables in vault under `concourse/`
- all common params used across all pipelines can be in `concourse/<TEAM_NAME>/` and pipeline specific params can be in `concourse/<TEAM_NAME>/<PIPELINE_NAME>`
- to write to vault the syntax is
  - `vault write concourse/<TEAM-NAME>/<PIPELINE-NAME>/<VAR_NAME> value=<VAR_VALUE>`
- ensure in the pipelines you use `((VAR_NAME))` instead of `{{VAR_NAME}}`
