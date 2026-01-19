# Regenerating the Demo SOPS Key

The `resources/dev-age.agekey` is a publicly known, insecure key provided for demonstration purposes to make this example work out-of-the-box. You should never commit your own private keys to a repository.

If for any reason you need to regenerate this demo key and the encrypted secrets, this guide provides the necessary steps.

## Steps to Regenerate

1.  **Generate a new key pair**: This command will overwrite the old demo key with a new one.
    ```bash
    age-keygen -o ./resources/dev-age.agekey
    ```
    The command will also print the new public key, which you will need in the next step.

2.  **Extract the Public Key**: If you didn't copy the public key from the previous step's output, you can extract it from the key file:
    ```bash
    export AGE_PUBLIC_KEY=$(grep "public key" ./resources/dev-age.agekey | awk '{print $4}')
    echo "New Public Key: ${AGE_PUBLIC_KEY}"
    ```

3.  **Re-encrypt all Secret Files**: You must now re-encrypt any files that were encrypted with the old key. In this project, that is `instance/secret.yaml`.
    ```bash
    sops --encrypt --age ${AGE_PUBLIC_KEY} \
        --encrypted-regex '^(data|stringData)$' \
        --in-place ./instance/secret.yaml
    ```
    *(`--in-place` modifies the file directly)*.

4.  **Commit the Changes**: Commit the updated `resources/dev-age.agekey` and the re-encrypted `instance/secret.yaml` to your Git repository so that Flux can use the new versions.
    ```bash
    git add ./resources/dev-age.agekey ./instance/secret.yaml
    git commit -m "chore: regenerate demo SOPS key and re-encrypt secrets"
    ```

Now the repository is updated with the new key and the secrets are encrypted accordingly.
