# Credentials Store Management Utility

`credentials_store` is the command line tool that stores the credentials the Seerene analysis tool chain
needs to reach your development infrastructure: Git hosting (GitHub, GitLab, Bitbucket, Azure DevOps),
issue trackers such as Jira, and code quality tools such as SonarQube. Credentials are kept in an
encrypted file named `.credentials_store` in the directory where the Seerene tools run, so they never
appear in plain text inside configuration files.

## Download

Each release on the [Releases](https://github.com/Seerene/credentials-store/releases) page ships:

| Asset | Platform |
|---|---|
| `credentials_store-linux-x86_64` | Linux, 64-bit Intel/AMD |
| `credentials_store-linux-arm64` | Linux, 64-bit ARM |
| `credentials_store-windows-x86_64.exe` | Windows, 64-bit, code-signed by Seerene GmbH |
| `SHA256SUMS` | Checksums of the three binaries |

The binaries are self-contained and need no runtime. On Linux, make the file executable:

```
chmod +x credentials_store-linux-x86_64
```

Verify the download before use:

```
sha256sum -c SHA256SUMS --ignore-missing
```

On Windows, right-click the `.exe`, open *Properties*, and confirm the *Digital Signatures* tab lists
Seerene GmbH.

## Usage

Run the tool from the directory that contains, or should contain, the `.credentials_store` file. That is
the working directory of the Seerene tools that will use the credentials.

| Command | Purpose |
|---|---|
| `add -k <key> -u <username> -p <password>` | Store or replace credentials under `<key>`. |
| `add -k <key>` | Same, prompting for username and password without echoing. |
| `add-json -j <file>` | Store every entry of a JSON list, see below. |
| `remove -k <key>` | Remove the credentials stored under `<key>`. |
| `list` | Print the stored keys. |
| `check -j <file>` | Verify that the store matches every entry of a JSON list. |
| `print_version` | Print the version. |

`<key>` is the name you later reference in the Seerene configuration files. The password is usually a
personal access token or app password rather than a login password.

### Examples

```
credentials_store add --key jira_credentials      --username "your-email@example.com" --password "JIRATOKEN"
credentials_store add --key github_credentials    --username "your-github-username"   --password "GITHUBTOKEN"
credentials_store add --key gitlab_credentials    --username "your-gitlab-username"   --password "GITLABTOKEN"
credentials_store add --key azure_credentials     --username "your-email@example.com" --password "AZURETOKEN"
credentials_store add --key bitbucket_credentials --username "your-bitbucket-username" --password "APPPASSWORD"
```

For a service that needs no authentication, store a placeholder username without a password:

```
credentials_store add --key no-credentials --username "unauthenticated"
```

### JSON input for `add-json` and `check`

```json
[
  {"key": "jira_credentials",   "username": "your-email@example.com", "password": "JIRATOKEN"},
  {"key": "github_credentials", "username": "your-github-username",   "password": "GITHUBTOKEN"}
]
```

`check` exits with status 1 and lists every key that is missing or differs. It never prints passwords.

### Shell quoting

Quote values that contain spaces or characters your shell treats specially. On Windows use double quotes:

```
credentials_store-windows-x86_64.exe add --key "jira_credentials" --username "user\"name" --password "pass\"word"
```

Passwords passed on the command line are visible to other users of the same machine through the process
list. On shared machines prefer the interactive prompt or `add-json`.

## Security notes

- The store file is written atomically and, on Linux, readable only by its owner.
- Values are encrypted with AES-256-CBC. The key is shared by all Seerene tools so they can read the
  store; this protects against casual reading, not against an attacker with access to the file. Keep the
  file's permissions tight and do not commit it to version control.
- Each stored value uses a fresh random IV from the operating system's random number generator.

## Support

Contact your Seerene representative or [support@seerene.com](mailto:support@seerene.com).

---
Copyright 2026 Seerene GmbH, Potsdam, Germany.
