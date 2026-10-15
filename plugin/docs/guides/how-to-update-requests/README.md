# How to Update Requests

## Prerequisites

1. [Run the agent](../how-to-run-the-agent/)
2. [Configure the proxy](../proxy-configuration/)[ ](../proxy-configuration/)
3. [Record request(s)](../how-to-record-requests/)

## Why Update Requests?

At some point in time, API providers will change the contract of a request. For example, maybe a component in the request or its response needs to be updated. When this happens we should also update the Stoobly request.

## Updating Requests

{% hint style="warning" %}
Before updating a request, it is recommended to create a [snapshot](../../core-concepts/mock-api/snapshots.md) in case changes need to be reverted.
{% endhint %}

There are three main strategies to update request(s):

### Editing

* [Edit individual requests from the UI](editing-from-the-ui.md)
* [Edit request snapshots](editing-with-snapshots.md)

### Replaying

* [Replay individual requests](../how-to-replay-requests/)
* [Replay scenarios](how-to-update-scenarios/updating-with-replay.md)

### Overwriting

* [Overwriting scenarios](how-to-update-scenarios/updating-with-overwrite.md)

## Persisting Changes

{% content-ref url="../how-to-mock-apis/how-to-snapshot-requests/" %}
[how-to-snapshot-requests](../how-to-mock-apis/how-to-snapshot-requests/)
{% endcontent-ref %}

## Reverting Changes

In case you update a request with unintended consequences, you can reset it to the last snapshot state. For more information see our documentation on the snapshots feature:

{% content-ref url="../../core-concepts/mock-api/snapshots.md" %}
[snapshots.md](../../core-concepts/mock-api/snapshots.md)
{% endcontent-ref %}

### Command

```bash
stoobly-agent request reset "<REQUEST-KEY>"
```

### Options

```bash
$ stoobly-agent request reset --help

Usage: stoobly-agent request reset [OPTIONS] REQUEST_KEY

  Reset a request to its snapshot state

Options:
  --force     Toggles whether resources are hard deleted.
  -h, --help  Show this message and exit.
```
