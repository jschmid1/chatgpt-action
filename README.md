# ChatGPT actions

A collection of ChatGPT assistants, e.g., code viewer, labeler, assigner, etc.

- [ChatGPT actions](#chatgpt-actions)
  - [Overview](#overview)
    - [Why reinvent the wheel?](#why-reinvent-the-wheel)
    - [Features](#features)
  - [Usage](#usage)
    - [Configuration](#configuration)
      - [Environment variables](#environment-variables)
      - [Inputs](#inputs)
    - [Prompt templates:](#prompt-templates)
      - [Variables available in prompt templates](#variables-available-in-prompt-templates)
  - [Developing](#developing)
  - [FAQs](#faqs)
    - [Review pull request from forks](#review-pull-request-from-forks)
    - [Inspect the messages between ChatGPT server](#inspect-the-messages-between-chatgpt-server)
  - [License](#license)

## Overview

### Why reinvent the wheel?

There're already many [chatgpt-actions][1], why we need to reinvent the wheel?

- Leave too much comments on Github pull requests

  Some of existing actions send each hunks in a pull request diff to ChatGPT,
  and yields a comment (or comment snippet) for each hunk. That makes them in
  practically unusable, as receiving 20+ email notification after submitting
  a pull request is not a good experience. For the reviewer's perspective, we
  usually just focus on the hunks that are "too good" or "too bad".

  **This actions take such experience into consideration, and only leave comments
  for the hunks that are not "LGTM".** (We do have an option to leave comments
  for good hunks as well).

- Lack of customization of the prompt

  All of existing actions hard-coded the prompt template inside the source code
  and there's no chance to customize the prompt. When the built-in prompt is
  not good enough, ChatGPT responses with useless comments.

  At the same time, someone may have good prompt ideas, he/she want to try using
  ChatGPT as an AI assistant for code quality evaluation and review, but he/she
  needs to reinvent the wheel for a new action to play on Github.

  **This action makes the prompt template configurable.** With a set of variables
  like `$title`, `$description`, `$filename`, `$patch`, you can customize the
  message sent to ChatGPT for better experience.

  Prompt engineering is an _art_ to make ChatGPT success, we encourage the community
  to try discovering new, novel, and insightful prompts to make ChatGPT do
  code review job better, _**without reinventing the wheel for another chatgpt-action**_.

  **ANY SUGGESTIONS AND IMPROVEMENTS ARE HIGHLY APPRECIATED!!!**

### Features


- Code review your pull requests

  ```yaml
      - uses: jschmid1/chatgpt-action@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        with:
          debug: false
          action: review
          review_comment_lgtm: true
  ```

- Customizable prompt templates

  ```yaml
      - uses: jschmid1/chatgpt-action@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        with:
          debug: false
          action: review
          review_patch: |
            Hi ChatGPT, I have a pull request with title "$title" and the description is as follows,

            > $description

            This is the patch

            > $patch

            and here are the filenames:

            > $filename

            Do something custom with these
  ```

## Usage

```yaml
name: Code Review

permissions:
  contents: read
  pull-requests: write

on:
  pull_request:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          repository: ${{github.event.pull_request.head.repo.full_name}}
          ref: ${{github.event.pull_request.head.ref}}
          submodules: false

      - uses: jschmid1/chatgpt-action@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        with:
          debug: false
          action: review
          review_comment_lgtm: true
```

### Configuration

See also: [./action.yml](./action.yml)

#### Environment variables

- `GITHUB_TOKEN`
- `OPENAI_API_KEY`: use this to authenticate with OpenAI API, official ChatGPT's API.

#### Inputs

- `action`: The action to run, currently can be `review`
- `debug`: Enable debug mode, will show messages and responses between ChatGPT server in CI logs.
- `review_comment_lgtm`: Leave comments even the patch is LGTM
- `path_filters`: Rules to filter files to be reviewed.

### Prompt templates:

See also: [./action.yml](./action.yml)

- `review_beginning`: The beginning prompt of a code review dialog
- `review_patch`: The prompt for each chunks/patches

#### Variables available in prompt templates

- code review (`action: review`):

  - `$title`: Title of the pull requests.
  - `$description`: The description of the pull request.
  - `$filename`: Filename of the file being viewed.
  - `$patch`: The diff contents of the patch being viewed.

Any suggestions or pull requests for improving the prompts are highly appreciated.

## Developing

> First, you'll need to have a reasonably modern version of `node` handy, tested with node 16.

Install the dependencies

```bash
$ npm install
```

Build the typescript and package it for distribution

```bash
$ npm run build && npm run package
```

## FAQs

### Review pull request from forks

Github Actions limits the access of secrets from forked repositories. To enable
this feature, you need to use the `pull_request_target` event instead of
`pull_request` in your workflow file. Note that with `pull_request_target`, you
need extra configuration to ensure checking out the right commit:

```yaml
name: Code Review

permissions:
  contents: read
  pull-requests: write

on:
  pull_request_target:

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
        with:
          repository: ${{github.event.pull_request.head.repo.full_name}}
          ref: ${{github.event.pull_request.head.ref}}
          submodules: false

      - uses: jschmid1/chatgpt-action@main
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        with:
          debug: false
          action: review
          review_comment_lgtm: true
```

### Inspect the messages between ChatGPT server

Set `debug: true` in the workflow file to enable debug mode, which will show the messages

[1]: https://github.com/marketplace?type=&verification=&query=chatgpt-action+
[2]: https://www.npmjs.com/package/chatgpt

## License

The MIT License (MIT), Courtesy to Tao He for most of the base code and the inspiration.
