---
layout: post
title: "Creating Excellent Developer User Experiences with Github Actions"
date: 2022-05-31 14:00
comments: true
---

----

_Cross-post: original posted on the [Doximity Technology Blog](https://technology.doximity.com/articles/creating-excellent-developer-user-experiences-with-github-actions)_

Github Actions can be used to ensure developers get the information they need _when_ it is relevant to them. When building software at scale an ever challenging task is to ensure everyone in the team has acknowledged a change to engineering processes — communicating with such a broad audience and making sure the message has come across becomes hard and that's when Github Actions workflows come to the rescue.

Widely adopted approaches like notifying everyone in a Slack channel, sending an email to a group of people, or a Lunch and Learn, while valid, may easily be missed due to relevancy or people not seeing them. Even if they read the message or watch the Lunch and Learn, by the time that information is really needed, they might have completely forgotten about it due to the sheer amount of other activities they are involved with.

Let’s take a look at two use cases in which we were able to more efficiently communicate by leaning into this continuous integration platform:

## Use case #1: Bring awareness to a new continuous deployment process

At Doximity, we have a services-oriented architecture and use feature branch development. Our product developers are used to manually deploying applications to production by doing a 2-step process: merging their changes into master and then deploying via a Slack bot.

Months ago, we added a new component to our architecture: a GraphQL gateway. Differently than the manual deployment process used for other applications, deployment was managed by automation: whenever developers merge into their applications GraphQL-related changes, the gateway incorporates those changes and auto-deploys to production.

Because the deployment process went from being atomic to eventually consistent order now mattered: developers **had** to wait for both GraphQL gateway _and_ their application backends to be in production _before_ shipping client changes. Failure to do so would cause a Javascript frontend or a mobile application to perform invalid requests, breaking the experience for our end users.

> How could we communicate the process change efficiently so no developer would unknowingly cause production downtime?

Slack and Github are part of the day-to-day of our developers, and we already even had a mapping for Github usernames and Slack usernames, so we decided that a good way to notify developers was to message them directly on Slack whenever they merged GraphQL-related changes into their applications.

The message sent to developers looked like this:
![](https://res.cloudinary.com/dhttas9u5/image/upload/Screen_Shot_2022-05-26_at_3.33.32_PM_ed21i2.jpg)

Some big wins here were:
1. **Clear expectations:** developers know _if_ and _when_ the auto-deployment should occur and communication channels that may be used if they need help to troubleshoot.
2. **Education:** along with the awareness brought up by the notification we link to our wiki page that expands on this concept with break downs and diagrams.
3. **Communication Efficiency:** because we are sending a direct message at a crucial moment we have their full attention.

A slightly simplified version of that Github Actions workflow looks like this:

{%raw%}
```yaml
name: "Notify author on gateway auto-deployment"
on:
  push:
    branches:
      - master
      - main
jobs:
  verify_graphql_changes:
    runs-on: ubuntu-latest
    outputs:
      has_gql_changes: ${{ steps.graphql_changes_check.outputs.callback_return }}
    steps:
      - id: graphql_changes_check
        name: Check for GraphQL changes in master HEAD
        uses: doximity/gh-action-callback-list-files@v1.0.0
        with:
          ref: master
          callback: |
            return filenamesList.some((elem) => { return elem.match(/graphql|gql/) })
  notify_dev_on_slack:
    runs-on: ubuntu-latest
    needs: verify_graphql_changes
    if: needs.verify_graphql_changes.outputs.has_gql_changes == 'true'
    steps:
      - uses: actions/checkout@v2
      - name: Send Slack message to commit author
        run: ./.github/workflows/scripts/notify_dev_on_slack.sh
        env:
          COMMIT_GITHUB_USERNAME: ${{ github.actor }}
```
{%endraw%}

Some highlights there:

- The GraphQL check is performed via one of our open-sourced actions, [doximity/gh-action-callback-list-files](https://github.com/marketplace/actions/run-callback-on-list-of-filenames). It's a simple check that works by pulling a list of filenames belonging to the master branch's HEAD and verifying whether they include the term "graphql" or "gql".
- This workflow is triggered whenever a push to the master/main branches happens. Because we have branch protection rules set up, this event is only triggered when the developer presses "Merge pull request" from their pull request. Github Actions makes trivial for us to grab the triggering Github username via `${{ github.actor }}`

## Use case #2: Allow developers to force-publish GraphQL breaking changes

We previously had in place a very cumbersome process that product developers had to go through in order to be able to deploy GraphQL breaking changes.

For those unfamiliar with GraphQL breaking changes, it's a change that could (depending on the usage) result in client error. A good example would be removing a field from your GraphQL schema. In practice, you first mark the field as deprecated, and once you are confident no one is using it anymore, then you go ahead and decommission it. Good monitoring of the clients usage of such fields in production is crucial — fortunately, we have exceptional monitoring in place for our GraphQL schema.

The previous process consisted of having a developer SSHing into a continuous integration server in order to be able to run a series of CLI commands to force-publish them. Deploying a so-called breaking change is already a pretty stressful task on its own and making it more error-prone, by requiring a developer to type a command from a continuous integration box, wasn't a very good idea.

> How could we improve their experience as well as tighten up this process?

Our GraphQL schema safety check blocks merging due to it being required so we knew the developer would be looking at Github, therefore it made sense to choose a path that would integrate with their pull request workflows. The solution we ended up with was to have a commit with a specific message pushed to their branch in order to mark the changes as safe.

Whenever breaking changes are detected a bot comments on the pull request indicating next steps:
![](https://res.cloudinary.com/dhttas9u5/image/upload/Screen_Shot_2022-05-26_at_12.07.26_PM_zzpj4b.jpg)

This gave us a couple things:
1. **Integration and Education:** We let the developer know what is happening at the spot they will be looking, how they can proceed and an easy way to bypass the check if needed.
2. **Ease:** The developer can do this from their pull request.
3. **Audit:** We can see who made and authorized the change.

A simplified workflow implementation looks like this:

{%raw%}
```yaml
name: "Detect and notify on GraphQL breaking changes"
on: pull_request
jobs:
  verify_graphql_changes:
    runs-on: ubuntu-latest
    outputs:
      has_gql_changes: ${{ steps.graphql_changes_check.outputs.callback_return }}
    steps:
      - id: graphql_changes_check
        name: Check for GraphQL changes in the PR
        uses: doximity/gh-action-callback-list-files@v1.0.0
        with:
          pr_number: ${{ github.event.pull_request.number }}
          callback: |
            return filenamesList.some((elem) => { return elem.match(/graphql|gql/) })
  validate_graphql_schema_changes:
    runs-on: ubuntu-latest
    needs: verify_graphql_changes
    if: needs.verify_graphql_changes.outputs.has_gql_changes == 'true'
    steps:
      - uses: actions/checkout@v2
        with:
          ref: ${{ github.event.pull_request.head.sha }}
      - id: validate_schema
        name: Validate GraphQL Schema
        continue-on-error: true
        run: ./.github/workflows/scripts/validate_graphql_schema.sh
        env:
          TRIGGERING_COMMIT_SHA1: ${{ github.event.after }}
          COMMIT_MESSAGE_TO_CONSENT: "Mark breaking changes as safe"
      - id: comment_on_breaking_changes
        name: PR comment on breaking changes
        uses: thollander/actions-comment-pull-request@v1
        if: ${{ steps.validate_schema.outcome == 'failure' }}
        with:
          message: |
            GraphQL breaking changes were detected. You _must_ confirm before considering merging it into master.
            It is *IMPERATIVE* that you are absolutely sure your changes are safe before bypassing this check.
            To proceed you have to either:
              - fix the conflict
              - push to this branch a commit with the message "Mark breaking changes as safe", i.e. `git commit --allow-empty -m "Mark breaking changes as safe"`
      - name: Signal failure after commenting on breaking changes
        if: ${{ steps.comment_on_breaking_changes.outcome == 'success' }}
        run: |
          echo "Breaking changes detected. Failing the build."
          exit 1
```
{%endraw%}

Some highlights there:

- Once again, we make use of [doximity/gh-action-callback-list-files](https://github.com/marketplace/actions/run-callback-on-list-of-filenames) to check for GraphQL-related changes. This time instead of checking against master's HEAD we pass the pull request number via `${{ github.event.pull_request.number }}`
- In the `validate_graphql_schema_changes` job, right after we checkout the pull request branch, our script `validate_graphql_schema.sh` verifies if the message for the workflow triggering commit (provided by Github Actions via `${{ github.event.after }}`) matches the environment variable `COMMIT_MESSAGE_TO_CONSENT`. If it does then that Shell script returns a successful exit code (`exit 0`) and the workflow moves on — that's how developers mark breaking changes as safe.
- Commenting on the pull request is a plug-n-play behavior that's handled by a third-party Github action. The only custom logic we have around it is to run that step only when the `validate_schema` step fails.

## Final Remarks

I hope with this post I was able to demonstrate how bringing some automation into the picture can make some significant positive impact on the quality of life for developers in an organization, aid in improving the communication within your teams, and make certain engineering processes less brittle.

Another worthy mention is that recently Github also released the ability for workflows to be shared across internal repositories within an enterprise account meaning that you can encapsulate the complex logic of a workflow from a repository upstream and whenever it changes every other repository referencing it will immediately reflect the changes.

We have found that there isn't a very steep learning curve to getting into Github Actions and you should start producing workflows like the ones above in no time.

----

Special thanks goes out to Bruno Miranda, Austin Story and Yuta Morinaga for reading drafts of this blog post, and for giving me their feedback!

Be sure to follow [@doximity_tech](https://twitter.com/doximity_tech) if you'd like to be notified about new blog posts.
