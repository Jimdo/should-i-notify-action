# Should I notify?
A GitHub action to decide if a status message should be pushed to slack for this build

**WARNING:** This action sends NO notification!

## How it makes the decision

1. If the current build failed, then it will recommend sending the message
2. If the current build succeeded, and the previous one failed, then it will recommend sending the message
3. If the current build succeeded, and the previous one succeeded, then it will recommend NOT sending the message

## Inputs

### needs_context

**required** json representation of the `needs` object in your context (`toJson(needs)`)

### branch

**required** The branch to get the status from

### github_token

**required** A github token (secrets.GITHUB_TOKEN will suffice)

## Outputs

### should_send_message

`yes|no` depending on if the message should be sent

## How to use

In this case, the action will take into account the result of `job1` and `job2` for builds only for the `main` branch. 

```yaml
job1:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v2
    - name: Run the script
      run: ./script1.sh
job2:
  runs-on: ubuntu-latest
  steps:
    - uses: actions/checkout@v2
    - name: Run the script
      run: ./script2.sh
      
slack-workflow-status:
  name: Post workflow status To Slack
  needs:
    - job1
    - job2
  if: always() && github.ref == 'refs/heads/main'
  runs-on: ubuntu-latest
  steps:
    - name: Determine if we need to notify
      uses: Jimdo/should-i-notify-action@main
      id: should_notify
      with:
        branch: main
        needs_context: ${{ toJson(needs) }}
        github_token: ${{ secrets.GITHUB_TOKEN }}

    - name: Slack workflow notification
      if: steps.should_notify.outputs.should_send_message == 'yes'
      uses: Gamesight/slack-workflow-status@master
      with:
        repo_token: ${{secrets.GITHUB_TOKEN}}
        slack_webhook_url: 'https://hooks.slack.com/services/...'
        channel: 'notifications-slack-channel'
        name: 'Your great build bot'
```
