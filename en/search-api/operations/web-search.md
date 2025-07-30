---
title: How to perform text search in {{ search-api-full-name }} via API v2 in deferred mode
description: Follow this guide to learn how to use {{ search-api-name }}'s API v2 interface to send search queries and get search results in XML or HTML format in deferred (asynchronous) mode.
---

# Performing text search queries in deferred mode via API v2

With {{ search-api-name }}'s [API v2](../concepts/index.md#api-v2), you can perform text search through the Yandex search database and get search results in [XML](../concepts/response.md) or [HTML](../concepts/html-response.md) format in deferred (asynchronous) mode. You can run queries using [REST API](../api-ref/) and [gRPC API](../api-ref/grpc/). The search results you get depend on the parameters specified in your query.

## Getting started {#before-you-begin}

{% include [before-begin](../../_tutorials/_tutorials_includes/before-you-begin.md) %}

To use the examples, install the [cURL](https://curl.haxx.se) and [jq](https://stedolan.github.io/jq) utilities, plus [gRPCurl](https://github.com/fullstorydev/grpcurl) if you are going to use [gRPC API](../api-ref/grpc/).

## Get your cloud ready {#initial-setup}

{% include [prepare-cloud-v2](../../_includes/search-api/prepare-cloud-v2.md) %}

## Create a search query {#form-request}

{% list tabs group=instructions %}

- REST API {#api}

  1. Create a file with the request body, e.g., `body.json`:

      **body.json**

      {% include [http-body-v2](../../_includes/search-api/http-body-v2.md) %}

      {% cut "Description of fields" %}

      {% include [http-v2-body-params](../../_includes/search-api/http-v2-body-params.md) %}

      {% endcut %}

  1. Run an http request by specifying the IAM token you got earlier:

      ```bash
      curl \
        --request POST \
        --header "Authorization: Bearer <IAM_token>" \
        --data "@body.json" \
        "https://searchapi.{{ api-host }}/v2/web/searchAsync"
      ```

      Result:

      ```text
      {
       "done": false,
       "id": "sppger465oq1********",
       "description": "WEB search async",
       "createdAt": "2024-10-02T19:51:02Z",
       "createdBy": "bfbud0oddqp4********",
       "modifiedAt": "2024-10-02T19:51:03Z"
      }
      ```

- gRPC API {#grpc-api}

  1. Create a file with the request body, e.g., `body.json`:

      **body.json**

      {% include [grpc-body-v2](../../_includes/search-api/grpc-body-v2.md) %}

      {% cut "Description of fields" %}

      {% include [grpc-v2-body-params](../../_includes/search-api/grpc-v2-body-params.md) %}

      {% endcut %}

  1. Run a gRPC call by specifying the IAM token you got earlier:

      ```bash
      grpcurl \
        -rpc-header "Authorization: Bearer <IAM_token>" \
        -d @ < body.json \
        searchapi.{{ api-host }}:443 yandex.cloud.searchapi.v2.WebSearchAsyncService/Search
      ```

      Result:

      ```text
      {
        "id": "spp3gp3vhna6********",
        "description": "WEB search async",
        "createdAt": "2024-10-02T19:14:41Z",
        "createdBy": "bfbud0oddqp4********",
        "modifiedAt": "2024-10-02T19:14:42Z"
      }
      ```

{% endlist %}

Save the obtained [Operation object](../../api-design-guide/concepts/operation.md) ID (`id` value) for later use.

## Make sure the request was executed successfully {#verify-operation}

Wait until {{ search-api-name }} executes the request and generates a response. This may take from five minutes to a few hours.

Make sure the request was executed successfully:

{% list tabs group=instructions %}

- REST API {#api}

  Run an http request:

  ```bash
  curl \
    --request GET \
    --header "Authorization: Bearer <IAM_token>" \
    https://operation.{{ api-host }}/operations/<request_ID>
  ```

  Where:

  * `<IAM_token>`: Previously obtained IAM token.
  * `<request_ID>`: The Operation object ID you saved at the previous step.

  Result:

  ```text
  {
   "done": true,
   "response": {
    "@type": "type.googleapis.com/yandex.cloud.searchapi.v2.WebSearchResponse",
    "rawData": "<Base64_encoded_XML_response_body>"
   },
   "id": "spp82pc07ebl********",
   "description": "WEB search async",
   "createdAt": "2024-10-03T08:07:07Z",
   "createdBy": "bfbud0oddqp4********",
   "modifiedAt": "2024-10-03T08:12:09Z"
  }
  ```

- gRPC API {#grpc-api}

  Run this gRPC call:

  ```bash
  grpcurl \
    -rpc-header "Authorization: Bearer <IAM_token>" \
    -d '{"operation_id": "<request_ID>"}' \
    operation.{{ api-host }}:443 yandex.cloud.operation.OperationService/Get
  ```

  Where:

  * `<IAM_token>`: Previously obtained IAM token.
  * `<request_ID>`: The Operation object ID you saved at the previous step.

  Result:

  ```text
  {
    "id": "spp82pc07ebl********",
    "description": "WEB search async",
    "createdAt": "2024-10-03T08:07:07Z",
    "createdBy": "bfbud0oddqp4********",
    "modifiedAt": "2024-10-03T08:12:09Z",
    "done": true,
    "response": {
      "@type": "type.googleapis.com/yandex.cloud.searchapi.v2.WebSearchResponse",
      "rawData": "<Base64_encoded_XML_response_body>"
    }
  }
  ```

{% endlist %}

If the `done` field is set to `true` and the `response` object is present in the output, the request has been completed successfully, so you can move on to the next step. Otherwise, repeat the check later.

## Get a response {#get-response}

After {{ search-api-name }} has successfully processed the request:

1. Get the result:

    {% list tabs group=instructions %}

    - REST API {#api}

      ```bash
      curl \
        --request GET \
        --header "Authorization: Bearer <IAM_token>" \
        https://operation.{{ api-host }}/operations/<request_ID> \
        > result.json
      ```

    - gRPC API {#grpc-api}

      ```bash
      grpcurl \
        -rpc-header "Authorization: Bearer <IAM_token>" \
        -d '{"operation_id": "<request_ID>"}' \
        operation.{{ api-host }}:443 yandex.cloud.operation.OperationService/Get \
        > result.json
      ```

    {% endlist %}

    Eventually the search query result will be saved to a file named `result.json` containing a [Base64-encoded](https://en.wikipedia.org/wiki/Base64) [XML](../concepts/response.md) or [HTML](../concepts/html-response.md) response in the `response.rawData` field.

1. Depending on the requested response format, decode the result from `Base64`:

    {% list tabs group=search_api_request %}

    - XML {#xml}

      ```bash
      echo "$(< result.json)" | \
        jq -r .response.rawData | \
        base64 --decode > result.xml
      ```

      The XML response to the query will be saved to a file named `result.xml`.

    - HTML {#html}

      ```bash
      echo "$(< result.json)" | \
        jq -r .response.rawData | \
        base64 --decode > result.html
      ```

      The HTML response to the query will be saved to a file named `result.html`.

    {% endlist %}

#### See also {#see-also}

* [{#T}](./web-search-sync.md)
* [{#T}](../concepts/web-search.md)
* [{#T}](../api-ref/authentication.md)
* [{#T}](../concepts/response.md)
* [{#T}](../concepts/html-response.md)