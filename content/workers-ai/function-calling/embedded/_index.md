---
pcx_content_type: concept
weight: 2
title: Embedded
meta:
  title: Embedded function calling
---

{{<heading-pill style="beta">}} Embedded function calling {{</heading-pill>}}

Cloudflare has a unique [embedded function calling](https://blog.cloudflare.com/embedded-function-calling) feature that allows you to execute function code alongside your tool call inference. Our npm package [`@cloudflare/ai-utils`](https://www.npmjs.com/package/@cloudflare/ai-utils) is the developer toolkit to get started.

Embedded function calling can be used to easily make complex agents that interact with websites and APIs, like using natural language to create meetings on Google Calendar, saving data to Notion, automatically routing requests to other APIs, saving data to an R2 bucket - or all of this at the same time. All you need is a prompt and an OpenAPI spec to get started.


## Get Started

To get started, run the following command in your project repository.

```sh
$ npm install @cloudflare/ai-utils --save
```

Then, import the utils with `import { createToolsFromOpenAPISpec, runWithTools, autoTrimTools } from "@cloudflare/ai-utils"` in your `index.js` file.

Check out the working example and API reference below. Our `ai-utils package` is also open-sourced on [Github](https://github.com/cloudflare/ai-utils) with a helpful [quickstart example](https://github.com/cloudflare/ai-utils/tree/main/examples/quickstart).

```js
---
header: Embedded function calling example
---
// make sure you run `npm install @cloudflare/ai-utils`
import { createToolsFromOpenAPISpec, runWithTools, autoTrimTools } from "@cloudflare/ai-utils"

export default {
  async fetch(request, env, ctx) {

    const response = await runWithTools(
      env.AI, // ai binding
      "@hf/nousresearch/hermes-2-pro-mistral-7b", // model that supports function calling
      {
        messages: [
          {
            role: "user",
            content: "Who is Cloudflare on github?"
          }
        ],
        tools: [
            // method that automatically generates the tool schemas given an OpenAPI spec
            ...await createToolsFromOpenAPISpec(
                // This is a gist of a partial OpenAPI spec, as the original one was too long
                'https://gist.githubusercontent.com/mchenco/fd8f20c8f06d50af40b94b0671273dc1/raw/f9d4b5cd5944cc32d6b34cad0406d96fd3acaca6/partial_api.github.com.json',
                { overrides: [{
                    matcher: ({ url, method }) => {
                        return url.hostname === "api.github.com"
                    },
                    // Github API requires a User-Agent header, which we can add with an override
                    values: {
                        headers: {"User-Agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/112.0.0.0 Safari/537.36",}
                    }
                }]}
            ),
        ]
      },   
      {
        // strictValidation: true,
        // streamFinalResponse: true, 
        verbose: true, // for better logging
        trimFunction: autoTrimTools, // to automatically choose the right tools
      }
    ).then((response) => {
      return response
    })

    return new Response(JSON.stringify(response));
  }
}
```

## API reference

For more details, refer to [API reference](/workers-ai/function-calling/embedded/api-reference/).

## Troubleshooting

If you are getting a `BadInput` error, your inputs may exceed our current context window for our models. Try reducing input tokens to resolve this error.