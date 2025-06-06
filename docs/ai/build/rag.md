# RAG

By integrating RAG (Retrieval-Augmented Generation) files, your AI Apps can leverage domain-specific knowledge efficiently. With initial support for LanceDB and future compatibility with other vector databases, developers can enhance their applications' performance and accuracy. Additionally, publishing to IPFS ensures data integrity and accessibility.

Retrieval Augmented Generation (RAG) allows developers to provide a knowledge base outside of the LLMs training data. This means that the LLM can provide specific information about a dataset or have expertise in a certain area.

## Defining RAG

We provide an off the shelf tool to create datasets from markdown files and web sources. This tool extracts the data into chunks, generates embedding vectors and stores them in a vector DB.

Currently only [Lance DB](https://lancedb.github.io/lancedb/) is supported. You can [review Lance DB's documentation](https://lancedb.github.io/lancedb/basic/) to determine the best way to ingest and embed your chosen RAG source data.

```shell
subql-ai embed -i ./path/to/dir/with/markdown -o ./db --table your-table-name --model nomic-embed-text
```

RAG data can also be defined with your own tooling.

::: info

You can follow through a step by step tutorial on how parse, vectorise, and add the resulting RAG database to your AI App in our [RAG quick start guide](../guides/subquery-docs-rag.md).

:::

## Updating RAG

As the source content changes, you may need to update your RAG data. To do this you can run the same command as used to define your initial RAG data. The tool will be able to determine changes to content and update the DB to match the changes, this makes the process much faster and reduces calls to the LLM.

## Adding RAG to your app

Once you have defined your RAG dataset you need to include it in your project.

First you will need to add it to your project manifest:

```ts
const project: ProjectManifest = {
  // ..The rest of your project manifest
  vectorStorage: {
    type: "lancedb",
    path: "./data.lance",
  },
  // Set this to the same model you use to generate your RAG db
  embeddingsModel: "nomic-embed-text",
};
```

In order for your project to be able to use this data you will also need to define a tool to consume it. We provide a built in RagTool which you can use, if you need more specific functionality you can extend this or build your own.

```ts
import { RagTool } from "jsr:@subql/ai-app-framework";

// Add this to your array of tools in your project,
// The first argument is your table name and the second is the column you want to select
new RagTool("subql-docs", "content");
```

::: details Tool implementation

```ts
export class RagTool extends FunctionTool {
  /**
   * RagTool is a default implementation allowing querying RAG data
   * @param tableName The name of the table to query
   * @param column The column on the table to extract results from
   */
  constructor(readonly tableName: string, readonly column: string) {
    super();
  }

  get description(): string {
    return `This tool gets relevant information from the ${this.tableName}. It returns a list of results separated by newlines.`;
  }

  parameters = {
    type: "object",
    required: ["query"],
    properties: {
      account: {
        type: "string",
        description: "A search string, generally the users prompt",
      },
    },
  };

  async call({ query }: { query: string }, ctx: IContext): Promise<string> {
    const vector = await ctx.computeQueryEmbedding(query);
    const raw = await ctx.vectorSearch(this.tableName, vector);

    const res = raw
      .map((r) => r[this.column])
      .filter((c) => !!c)
      .join("\n");

    return res;
  }
}
```

:::

This tool does a few things:

1. Converts the user query input into an embedded vector data.
2. Searches the specified vector db table for the closest matches.
3. Processes the raw results. In this example the data has a column called `content` that needs to be extracted and combined for the LLM to use as the answer.

#### Add your tool to your project

```ts
const tools: FunctionTool[] = [
  // ..The rest of your tools
  new SubqueryDocs(),
];
```

## Updating your projects data

RAG data is generally not static, the source of this information can change and evolve.
That means from time to time it is handy to rebuild your database.

In order to do this when you define your application the path to your Lance DB should not use a local or IPFS path.
Instead you should use any of the supported storage options that [Lance DB Supports](https://lancedb.github.io/lancedb/concepts/storage/).

Example:

```ts

const project: ProjectManifest = {
  // ..The rest of your project manifest
  vectorStorage: {
    type: "lancedb",
    path: "s3://my-bucket/my-path/data.lance"
}
```
