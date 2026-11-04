# ShrinkML check

A GitHub Action that fails a pull request when a model version would not pass ShrinkML's promotion check.

It asks the same question the **Make active** button asks: is the version at least 80% on the frozen Holdout, and
strictly better than the version that is active now, scored on the same Holdout? It only asks. It never makes a
version active, never trains, and never changes anything in your project, so a **read-only API key is enough** and is
the key to use here.

## Use it

```yaml
name: ShrinkML check
on: [pull_request]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: teyrallc/shrinkml-check-action@v1
        with:
          project-id: 0f6c1d2e-0000-4000-8000-000000000000 # the uuid in your project page's address
          api-key: ${{ secrets.SHRINKML_API_KEY }} # a read-only key
          candidate: model/ # the Package folder (or zip) your pull request changes, or a version such as v3
```

The job turns red when the version would not pass, and says why in one sentence. The same sentence goes to the job
summary.

## Inputs

| Input           | Needed | What it is                                                                                                                                                     |
| --------------- | ------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `project-id`    | yes    | The project's id.                                                                                                                                              |
| `api-key`       | yes    | A ShrinkML API key. Create a read-only one in **Settings, API Keys**, store it as a repository secret, and pass the secret. Never write the key into the file. |
| `candidate`     | yes    | The version to check: a Package folder or zip (as `shrinkml package pull` leaves it), a version number such as `v3`, or a version id.                          |
| `baseline`      | no     | A baseline file to compare with as well. By default `shrinkml.baseline.json` in the working folder is used when it exists.                                     |
| `baseline-only` | no     | `true` compares with the baseline file only, not with the active version.                                                                                      |
| `base-url`      | no     | The ShrinkML API address, if you do not use the default.                                                                                                       |
| `language`      | no     | `en` (default) or `zh-Hant`, for the words in the log.                                                                                                         |
| `cli-package`   | no     | What to install the command-line tool from (default `shrinkml-cli`). A path or a URL to a wheel also works.                                                    |

## Outputs

| Output        | What it is                                                                                        |
| ------------- | ------------------------------------------------------------------------------------------------- |
| `passed`      | `true` or `false`.                                                                                |
| `result-file` | The JSON the check wrote, the same shape as `shrinkml check --json` (schema `shrinkml.check.v1`). |

## How a Package names its version

A Package's `manifest.json` holds no score. The one thing in it that ties it to a version is the feedback key
(`feedback.key_id`) every Package made since the feedback loop carries. The check sends that key to ShrinkML, which
finds the version that made it **in your project only**. A Package from another project, or an older Package without a
key, is not guessed at: use `candidate: v3` instead.

## The baseline file

`shrinkml check --write-baseline` writes `shrinkml.baseline.json`, the active version's score on its Holdout. Commit it
with your code. From then on a pull request must beat that score too, and a new Holdout is a decision you make by
writing the file again, never something a version can slip past. This is not the Scorecard's **Baseline** (the
no-training reference); it is the score your team agreed to keep beating.

## Exit codes

The job fails for any exit code that is not 0. `0` passes, `1` does not pass or ShrinkML refused (for example, no such
version), `2` the Action was used wrongly (for example, no candidate named).

## Licence

Apache-2.0. See `LICENSE`.
