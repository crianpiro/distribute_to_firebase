# Distribute to Firebase App Distribution

A GitHub Action that uploads your Android or iOS artifacts (`.apk`, `.aab`, `.ipa`) to
[Firebase App Distribution](https://firebase.google.com/docs/app-distribution) in seconds.

It wraps the official Firebase CLI (`firebase appdistribution:distribute`) and adds:

- Sensible handling of optional inputs (empty values are never forwarded to the CLI).
- Authentication via inline credentials **or** a credentials file.
- Outputs with the tester link, console link, and a direct binary download link.

## Usage

```yaml
- name: Distribute to Firebase App Distribution
  uses: crianpiro/distribute_to_firebase@v1
  with:
    artifact: ./app/build/outputs/apk/debug/app-debug.apk
    app-id: ${{ secrets.FIREBASE_APP_ID }}
    credentials: ${{ secrets.FIREBASE_CREDENTIALS }}
    groups: qa-team, beta-testers
    release-notes: "Nightly build from ${{ github.sha }}"
```

### Capturing the release link

```yaml
- name: Distribute to Firebase App Distribution
  id: distribute
  uses: crianpiro/distribute_to_firebase@v1
  with:
    artifact: ./build/app.aab
    app-id: ${{ secrets.FIREBASE_APP_ID }}
    credentials: ${{ secrets.FIREBASE_CREDENTIALS }}
    testers: alice@example.com, bob@example.com

- name: Comment the link
  run: echo "Install it here: ${{ steps.distribute.outputs.release-link }}"
```

### Using a credentials file instead of inline contents

```yaml
- uses: crianpiro/distribute_to_firebase@v1
  with:
    artifact: ./build/app.ipa
    app-id: ${{ secrets.FIREBASE_APP_ID }}
    credentials-file: ${{ runner.temp }}/service-account.json
    groups-file: ./distribution/groups.txt
    release-notes-file: ./distribution/release-notes.md
```

## Inputs

| Name                 | Required | Description                                                                                  |
| -------------------- | :------: | -------------------------------------------------------------------------------------------- |
| `artifact`           |   yes    | Path to the artifact (`.apk`, `.aab` or `.ipa`) to distribute.                               |
| `app-id`             |   yes    | The Firebase App ID of the app you want to distribute.                                        |
| `credentials`        |  one of  | Contents of the Google service account credentials JSON. Pass via a secret.                  |
| `credentials-file`   |  one of  | Path to the Google service account credentials JSON file.                                     |
| `release-notes`      |    no    | Release notes for this release.                                                              |
| `release-notes-file` |    no    | Path to a file containing the release notes.                                                 |
| `testers`            |    no    | Comma-separated tester email addresses to invite.                                            |
| `testers-file`       |    no    | Path to a file containing tester email addresses.                                            |
| `groups`             |    no    | Comma-separated tester group aliases to share this release with.                             |
| `groups-file`        |    no    | Path to a file containing tester group aliases.                                              |

> You must provide **either** `credentials` **or** `credentials-file`.

## Outputs

| Name                   | Description                                                                          |
| ---------------------- | ------------------------------------------------------------------------------------ |
| `release-link`         | Tester-facing link to view release notes and install the app (`testingUri`).         |
| `console-link`         | Link to the release in the Firebase console (`firebaseConsoleUri`).                  |
| `binary-download-link` | Signed link that directly downloads the binary. Expires after one hour.              |

## Getting the credentials

1. In the [Google Cloud Console](https://console.cloud.google.com/iam-admin/serviceaccounts),
   create (or reuse) a service account with the **Firebase App Distribution Admin** role.
2. Create a JSON key for that service account and download it.
3. Add the file contents as a repository secret, e.g. `FIREBASE_CREDENTIALS`.
4. Add your Firebase App ID as a secret, e.g. `FIREBASE_APP_ID`.

See the [Firebase CLI distribution docs](https://firebase.google.com/docs/app-distribution/android/distribute-cli)
for more detail.

## Full workflow example

```yaml
name: Distribute debug build
on:
  push:
    branches: [main]

jobs:
  distribute:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      # ... build your app and produce the artifact ...

      - name: Distribute to Firebase App Distribution
        uses: crianpiro/distribute_to_firebase@v1
        with:
          artifact: ./app/build/outputs/apk/debug/app-debug.apk
          app-id: ${{ secrets.FIREBASE_APP_ID }}
          credentials: ${{ secrets.FIREBASE_CREDENTIALS }}
          groups: qa-team
          release-notes: ${{ github.event.head_commit.message }}
```

## License

[BSD 3-Clause](./LICENSE) © Cristian Andres Picon Rodriguez
