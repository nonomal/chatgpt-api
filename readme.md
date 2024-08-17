<p align="center">
  <a href="https://agentic.so">
    <img alt="Agentic" src="/docs/media/agentic-header.jpg" width="308">
  </a>
</p>

<p align="center">
  <em>AI agent stdlib that works with any LLM and TypeScript AI SDK.</em>
</p>

<p align="center">
  <a href="https://github.com/transitive-bullshit/agentic/actions/workflows/main.yml"><img alt="Build Status" src="https://github.com/transitive-bullshit/agentic/actions/workflows/main.yml/badge.svg" /></a>
  <a href="https://www.npmjs.com/package/@agentic/stdlib"><img alt="NPM" src="https://img.shields.io/npm/v/@agentic/stdlib.svg" /></a>
  <a href="https://github.com/transitive-bullshit/agentic/blob/main/license"><img alt="MIT License" src="https://img.shields.io/badge/license-MIT-blue" /></a>
  <a href="https://prettier.io"><img alt="Prettier Code Formatting" src="https://img.shields.io/badge/code_style-prettier-brightgreen.svg" /></a>
</p>

# Agentic <!-- omit from toc -->

- [Intro](#intro)
- [Docs](#docs)
- [Contributors](#contributors)
- [License](#license)

## Intro

Agentic is a **standard library of AI functions / tools** which are **optimized for both normal TS-usage as well as LLM-based usage**. Agentic works with all of the major TS AI SDKs (LangChain, LlamaIndex, Vercel AI SDK, OpenAI SDK, etc).

Agentic clients like `WeatherClient` can be used as normal TS classes:

```ts
import { WeatherClient } from '@agentic/stdlib'

// Requires `process.env.WEATHER_API_KEY` (free from weatherapi.com)
const weather = new WeatherClient()

const result = await weather.getCurrentWeather({
  q: 'San Francisco'
})
console.log(result)
```

Or you can use these clients as **LLM-based tools** where the LLM decides when and how to invoke the underlying functions for you.

This works across all of the major AI SDKs via adapters. Here's an example using [Vercel's AI SDK](https://github.com/vercel/ai):

```ts
// sdk-specific imports
import { openai } from '@ai-sdk/openai'
import { generateText } from 'ai'
import { createAISDKTools } from '@agentic/ai-sdk'

// sdk-agnostic imports
import { WeatherClient } from '@agentic/stdlib'

const weather = new WeatherClient()

const result = await generateText({
  model: openai('gpt-4o-mini'),
  // this is the key line which uses the `@agentic/ai-sdk` adapter
  tools: createAISDKTools(weather),
  toolChoice: 'required',
  prompt: 'What is the weather in San Francisco?'
})

console.log(result.toolResults[0])
```

You can use our standard library of thoroughly tested AI functions with your favorite AI SDK – without having to write any glue code!

Here's a slightly more complex example which uses multiple clients and selects a subset of their functions using the `AIFunctionSet.pick` method:

```ts
// sdk-specific imports
import { ChatModel, createAIRunner } from '@dexaai/dexter'
import { createDexterFunctions } from '@agentic/dexter'

// sdk-agnostic imports
import { PerigonClient, SerperClient } from '@agentic/stdlib'

async function main() {
  // Perigon is a news API and Serper is a Google search API
  const perigon = new PerigonClient()
  const serper = new SerperClient()

  const runner = createAIRunner({
    chatModel: new ChatModel({
      params: { model: 'gpt-4o-mini', temperature: 0 }
    }),
    functions: createDexterFunctions(
      perigon.functions.pick('search_news_stories'),
      serper
    ),
    systemMessage: 'You are a helpful assistant. Be as concise as possible.'
  })

  const result = await runner(
    'Summarize the latest news stories about the upcoming US election.'
  )
  console.log(result)
}
```

Here we've exposed 2 functions to the LLM, `search_news_stories` (which comes from the `PerigonClient.searchStories` method) and `serper_google_search` (which implicitly comes from the `SerperClient.search` method).

All of the SDK adapters like `createDexterFunctions` accept very flexible `AIFunctionLike` objects, which include:

- `AIFunctionSet` - Sets of AI functions (like `perigon.functions.pick('search_news_stories')` or `perigon.functions` or `serper.functions`)
- `AIFunctionsProvider` - Client classes which expose an `AIFunctionSet` via the `.functions` property (like `perigon` or `serper`)
- `AIFunction` - Individual functions (like `perigon.functions.get('search_news_stories')` or `serper.functions.get('serper_google_search')` or AI functions created directly via the `createAIFunction` utility function)

You can pass as many of these `AIFunctionLike` objects as you'd like and you can manipulate them as `AIFunctionSet` sets via `.pick`, `.omit`, `.get`, `.map`, etc.

## Docs

Full docs are available at [agentic.so](https://agentic.so).

## Contributors

- [Travis Fischer](https://x.com/transitive_bs)
- [David Zhang](https://x.com/dzhng)
- [Philipp Burckhardt](https://x.com/burckhap)
- And all of the [amazing OSS contributors](https://github.com/transitive-bullshit/agentic/graphs/contributors)!

## License

MIT © [Travis Fischer](https://x.com/transitive_bs)

To stay up to date or learn more, follow [@transitive_bs](https://x.com/transitive_bs) on Twitter.
