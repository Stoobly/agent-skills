# Troubleshooting

## Check Logs First

Before troubleshooting, always check the logs to see exactly what Stoobly intercepted and how it responded:

```bash
# Scaffold app — shows mock/record status per request
stoobly-agent scaffold request logs list mock

# Scaffold app — shows raw workflow process output
stoobly-agent scaffold workflow logs mock

# Standalone (no scaffold)
stoobly-agent request logs list
```

***

## After 499 Status Code (Mock Not Found)

{% stepper %}
{% step %}
**Double check if the correct scenario is used**

* Double check if request exists in the scenario
* By default the UI is configured to run on [http://localhost:4200](http://localhost:4200)
{% endstep %}

{% step %}
**Double check rewrite rules**

To learn more about rewrite rules, see [here](../../core-concepts/agent/proxy-settings/rewrite-rules.md)
{% endstep %}

{% step %}
**Double check lifecycle hooks**

To learn more about lifecycle hooks, see [here](../../core-concepts/agent/lifecycle-hooks.md)
{% endstep %}

{% step %}
**Double check match rules**

To learn more about match rules, see [here](../../core-concepts/agent/proxy-settings/match-rules.md)
{% endstep %}

{% step %}
**Test using CLI**

{% tabs %}
{% tab title="Mock Command" %}
CLI command to mock HTTP requests:

```
$ stoobly-agent mock --help
```

```
Usage: stoobly-agent mock [OPTIONS] URL

  Mock request

Options:
  -d, --data TEXT      HTTP POST data
  --format [raw]       Format response
  -H, --header TEXT    Pass custom header(s) to server
  -X, --request TEXT   Specify request command to use
  --scenario-key TEXT
  -h, --help           Show this message and exit.
```

Its usage is very similar to `cURL` where you specify a URL and flags with extra request data.

For example here's a GET request:

```
stoobly-agent mock https://google.com
```

For example here's a POST request

```
stoobly-agent mock -X POST https://my-service.com \
    --header "Content-Type: application/json" \
    --data '{"key": "value"}'
```
{% endtab %}

{% tab title="Response Show Command" %}
If you already have the request's key handy, you can use the `request response show` command

```
$ stoobly-agent request response show --help                  
```

```
Usage: stoobly-agent request response show [OPTIONS] REQUEST_KEY

  Retrieve mocked response

Options:
  -h, --help  Show this message and exit.
```

For example, if your Request key is "eyJwIjogMCwgImkiOiAzNjl9", the command would look like this:

```
$ stoobly-agent request response show eyJwIjogMCwgImkiOiAzNjl9
```

```
{"key": "value"}
```
{% endtab %}
{% endtabs %}
{% endstep %}
{% endstepper %}

## After Incorrect Response Returned

{% stepper %}
{% step %}
**Double check if the correct scenario is used**

* Double check if request exists in the scenario
* By default the UI is configured to run on [http://localhost:4200](http://localhost:4200)
{% endstep %}

{% step %}
**Create an Endpoint to enable component specific matching**

* To learn more, see [here](../../experimental/experimental-features/endpoints.md)
{% endstep %}
{% endstepper %}
