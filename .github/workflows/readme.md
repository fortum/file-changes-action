
# Workflow Information

- [Workflow Information](#workflow-information)
- [Overview](#overview)
  - [Schedule](#schedule)
  - [Issue Comment](#issue-comment)
  - [Pull Request](#pull-request)
  - [Push](#push)

# Overview

1. Make a **Pull Request** from your forked branch (forked from _master_) with changes to _trilom/file-changes-action/master_ branch.
2. Once merged into master this will lint the code and provide output in the checks, update the AUTHORS file, and package _dist/_.  If there is a release then this will create a **Pull Request** from the _v\*\*_ branch to _master_ and a comment will be made on the original **Pull Request** notifying contributors.  If there is not a release the changes will be **push**ed to _master_.
3. In the **Pull Request** linting and testing will be performed again.  If _linted_, _tested_, _builds_, and _lintdogged_ label exist and _hold merge_ does not the release will be merged into _master_.
4. Once merged this time [semantic-release](https://github.com/semantic-release/semantic-release) will run to create the Github Release, release notes, changelog, notify Slack, package and deploy to NPM and Github Package Repo, label the release, and notify any issues of it's deployment.
5. After user semantic-release-bot commits the release commit, this code will be pushed to the release branch.

## Schedule

- Everyday at 5:00 AM GMT:
  - Run integration tests.

## Issue Comment

- When any `created` **Issue Comment** type runs on a **Pull Request** from trilom with the body of `/release`(**automerge.yml**):
  - If _linted_, _tested_, _builds_, _lintdogged_, and _hold merge_ or _automated merge_ **does not** labels exist:
    - Merge the PR and add the _automated merge_ label
      - If failure, put some output on the original PR.

## Pull Request

- When any `opened`, `reopened`, or `synchronize` **Pull Request** type runs to the _master_ branch from a _v\*\*_ branch:
  - Run integration tests.

- When any `opened` or `reopened` **Pull Request** type runs on any branch other than _master_ from anyone other than trilom or trilom-bot from a forked branch(**close_pr.yml**):
  - Close the **Pull Request** and put the dunce cap on.

- When any `labeled`, or `closed` **Pull Request** type runs on _master_, _next_, _alpha_, or _beta_(**automerge.yml**):
  - If _linted_, _tested_, _builds_, _lintdogged_, and _hold merge_ or _automated merge_ **does not** labels exist:
    - Merge the PR and add the _automated merge_ label
      - If failure, put some output on the original PR.

- When any `opened`, `reopened`, or `synchronize` **Pull Request** type runs(**pr.yml**):
  - Assign it to trilom (**add-reviews**)
  - Build code with `make run` which runs `yarn` and `tsc` (**build**)
    - Label with builds if passing and on inner workspace
  - Test code with `make run COMMAND=test` which runs `jest` (**test**)
    - Label with tested if passing and on inner workspace
  - Test code with eslint reviewdog and report back if inner workspace (**lintdog**)
    - Label with pretty if passing and on inner workspace
  - Check format of code with `make run COMMAND=format-check` which runs `prettier --check` (**format_check_push**)
    - If:
      - Fork then pull **Pull Request** github.ref with GITHUB_TOKEN
      - Inner **Pull Request** then pull HEAD repo ref
    - Build code with `make run` which runs `yarn` and `tsc`
      - If format-check succeeds and on inner workspace
        - Label with pretty
      - If format-check fails and on inner workspace and actor is not trilom-bot
        - Run `make run COMMAND=format` which runs `prettier --write`
        - Clean build files with `make clean`
        - Commit the format changes as trilom-bot to **Pull Request** head

## Push

- When any **Push** type runs to _master_, _next_, _alpha_, or _beta_(**push.yml**):
  - Build code with `make run` which runs `yarn` and `tsc` (**build**)
  - Test code with `make run COMMAND=test` which runs `jest` (**test**)
  - Test code with eslint reviewdog and report back with github checks(**lintdog**)
- When any **Push** type runs to _master_, _next_, _alpha_, or _beta_ with a head_commit message **NOT** containing 'trilom/v1.' or 'trilom/v2.':
  - Build **dist/\*\*.js** files, update **AUTHORS**, format **src/\*\*.ts** files and commit.
  - Test [semantic-release](https://github.com/semantic-release/semantic-release) if a release is ready then create a **Pull Request**
    - Echo release outputs
    - Get changed files with [file-changes-action](https://github.com/trilom/file-changes-action) and build a message to post to new **Pull Request**
    - Comment on the original **Pull Request** with the new details of the release.
  - If no release, then **Push** changes directly back to master.
- When any **Push** type runs to _master_, _next_, _alpha_, or _beta_ with a head_commit message containing 'trilom/v1.' or 'trilom/v2.':
  - Run [semantic-release](https://github.com/semantic-release/semantic-release) to prepare Github Release, release notes, changelog, notify Slack, package and deploy to NPM and Github Package Repo, label the release, and notify any issues of it's deployment.
- When any **Push** type runs to _master_, _next_, _alpha_, or _beta_ from semantic-release-bot with a head_commit message containing 'chore(release):':
  - Get the **Pull Request** number from the **Push** and push the semantic-release changes to the tagged release branch.
