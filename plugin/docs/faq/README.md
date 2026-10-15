# FAQ

## How do I report an issue?

We track issues in Github at [https://github.com/Stoobly/stoobly-agent/issues](https://github.com/Stoobly/stoobly-agent/issues). Please provide as much information as possible and we will take a look :smile:

## Where do I find the latest releases?

pip releases can be found [on PyPI](https://pypi.org/project/stoobly-agent/).

Docker image releases can be found [on Docker Hub](https://hub.docker.com/r/stoobly/agent).

To currently see the agent code changes, see the [Github Releases](https://github.com/Stoobly/stoobly-agent/releases) and their changelogs.

To learn how the contributors publish releases, see our [Developer Guide on Releases](../developer-guide/releases.md).

## Is this a test framework?

While Stoobly provides similar functionality for running tests and providing test output, our primary goal is to **provide data storage for HTTP(s) requests** from which you can:

* Search requests
* Build requests into scenarios
* Generate endpoint contracts
* Mock requests
* Test request and scenarios

**At its core, Stoobly is an index** where the keys are the request components such as:

* URL, port number, path, headers, query parameters, body

And the returned value is the recorded request's response components:

* headers, body, status code

In regards to mocking and testing, we provide the ability to **dramatically reduce the need to write and maintain request and response components in test code**. Here is how we tackle the core challenges with testing:

1. Generation:
   * Record [requests](../core-concepts/mock-api/requests/) and [scenarios](../core-concepts/mock-api/scenarios.md)
2. Correctness:
   * Replay recorded requests and compare response against recorded response
3. Maintenance:
   * Replay requests to update responses
   * Contract testing to ensure validity of requests and responses

## Where do my requests get stored?

The Stoobly agent records and stores requests to your local machine. We use an underlying SQLite database file to store data. By default it is located at `~/.stoobly/db/stoobly_agent.sqlite3` for Unix systems.

## How much of a request is stored?

We store a HTTP request's - URL, port number, path, headers, query parameters, and body

And the HTTP response's - headers, body, and status code

Certain parts of the request may contain sensitive information. For example an [access token](https://en.wikipedia.org/wiki/Access_token) [passed in with a HTTP header](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Authorization) might be used for authorization. Another example could be username and password properties being passed in as part of the request body or as form data.

To address this concern, **we provide rewrite and filter rules for each part of the request**. See the following documentation:

## What are common terminologies?

<details>

<summary>Agent</summary>

This refers to the command-line utility `stoobly-agent`. See our [Core Concept Agent documentation](../core-concepts/agent/) for more information

</details>

<details>

<summary>CLI</summary>

This is the [command-line interface](https://en.wikipedia.org/wiki/Command-line_interface) that comes with `stoobly-agent` for interacting with Stoobly resources from a terminal or shell.

</details>

<details>

<summary>Proxy</summary>

This refers to the proxy server inside `stoobly-agent` responsible for intercepting, recording, replaying requests, etc.

</details>

<details>

<summary>Request</summary>

Unless specified, a request refers to a [Stoobly Request](../core-concepts/mock-api/requests/). Where applicable, the term "HTTP request" represents a request sent via the [HTTP Protocol](https://en.wikipedia.org/wiki/HTTP) to and from various sources and destinations.

</details>

<details>

<summary>Response</summary>

Unless specified, a response refers to a [Stoobly Response](../core-concepts/mock-api/requests/response.md).

</details>

<details>

<summary>Scenario</summary>

This refers to a collection of requests. See [Scenarios](../core-concepts/mock-api/scenarios.md) for more details.

</details>

<details>

<summary>UI or Dashboard</summary>

This refers to the `stoobly-agent`'s frontend component built with HTML, CSS and JavaScript.

</details>

<details>

<summary>499 HTTP Status Code</summary>

499 is a custom Stoobly client error response status code. It means that requests are successfully being sent through the `stoobly-agent`proxy. However, the request does not match any previously recorded requests by Stoobly.

This means that Stoobly is installed and working as expected, but there are issues with not found requests.

This occurs when the Mock Policy configuration is set to `all` . This will cause requests that have not been recorded yet to not be found resulting in the 499. If the setting is `found`, then any not found request will go to the remote destination automatically. See the Mock Policy section of [data-rules.md](../core-concepts/agent/proxy-settings/data-rules.md "mention")

To fix this, it is recommended to record the missing requests you're trying to mock.

</details>
