# LangChain RAG: Document Chunking, Vector Stores, Retrieval, and Agents

> Practical notes based on the lecture transcript and TypeScript
> examples. The examples use LangChain.js, OpenAI embeddings, and an
> in-memory vector store.

## 1. RAG pipeline overview

Retrieval-Augmented Generation (RAG) gives an LLM relevant information
from external documents at question time.

A typical ingestion and query flow is:

1.  **Load documents** from PDFs, DOCX files, or other supported
    sources.
2.  **Split documents** into smaller chunks.
3.  **Create embeddings** for each chunk.
4.  **Store chunks and vectors** in a vector store.
5.  **Embed the user's question** and retrieve relevant chunks.
6.  **Pass the retrieved context and question to the LLM** to generate
    an answer.

``` text
Source files
    |
    v
Document Loader
    |
    v
Document objects
    |
    v
RecursiveCharacterTextSplitter
    |
    v
Chunks
    |
    v
Embedding model
    |
    v
Vector store
    |
User question
    |
    v
Question embedding + retrieval
    |
    v
Relevant chunks
    |
    v
LLM + question + context
    |
    v
Grounded answer
```

The vector store performs retrieval; it does not normally compose the
final natural-language answer. The LLM uses the retrieved text to answer
the question.

------------------------------------------------------------------------

## 2. Chunk documents with `RecursiveCharacterTextSplitter`

A loader may return one `Document` per PDF page, or another document
structure depending on the loader. A page can still contain several
topics. Splitting it into smaller chunks can improve retrieval precision
and reduce the amount of context sent to the LLM.

Install the package:

``` bash
npm install @langchain/textsplitters
```

Import the splitter:

``` ts
import { RecursiveCharacterTextSplitter } from "@langchain/textsplitters";
```

Create a splitter:

``` ts
const textSplitter = new RecursiveCharacterTextSplitter({
  chunkSize: 1000,
  chunkOverlap: 200,
});

const allSplits = await textSplitter.splitDocuments(docs);

console.log("Chunks:", allSplits.length);
```

### Configuration properties

  ------------------------------------------------------------------------
  Property              Meaning                                    Example
  --------------------- --------------------- ----------------------------
  `chunkSize`           Target maximum size                         `1000`
                        of each chunk,        
                        measured in           
                        characters by default 

  `chunkOverlap`        Number of characters                         `200`
                        repeated between      
                        adjacent chunks       

  `separators`          Ordered boundaries       `["\n\n", "\n", " ", ""]`
                        the recursive         
                        splitter tries when   
                        splitting text        

  `lengthFunction`      Function used to      Defaults to character length
                        measure chunk length  

  `keepSeparator`       Controls whether             Version/API dependent
                        separators are        
                        retained with split   
                        text                  

  `addStartIndex`       Adds a chunk's                              `true`
                        starting character    
                        index to metadata     
  ------------------------------------------------------------------------

The most important starting parameters are `chunkSize` and
`chunkOverlap`. Check the installed package's TypeScript definitions for
the exact options supported by your version.

### What does `chunkSize: 1000` mean?

It asks the splitter to produce chunks of approximately 1,000 characters
or fewer, subject to the splitting strategy and separators. It does
**not** mean 1,000 tokens.

The recursive splitter tries to preserve natural text boundaries. It
generally attempts larger separators first (such as paragraph breaks),
then smaller ones (such as line breaks and spaces), and finally
character-level splitting if needed.

### What does `chunkOverlap: 200` mean?

Adjacent chunks share approximately 200 characters of text.

For example:

``` text
Chunk 1: [----------------------------------------]
Chunk 2:                             [----------------------------------------]
                                      <--- overlap --->
```

Overlap helps preserve context when a sentence, explanation, or topic
crosses a chunk boundary. Without overlap, the first part of an idea
could be in one chunk and its continuation in the next.

A larger overlap is not automatically better. It increases the amount of
duplicated text, storage, embedding work, and potentially redundant
retrieval results.

### Choosing chunk size and overlap

There is no universal best configuration. Start with a reasonable
baseline, evaluate retrieval quality against representative questions,
and tune it.

  -----------------------------------------------------------------------
  Situation                           Possible starting point
  ----------------------------------- -----------------------------------
  Short FAQ entries                   Smaller chunks, often 300--700
                                      characters

  General prose and reports           Around 800--1,200 characters

  Longer explanations with            Larger chunks may preserve more
  dependencies                        context

  Dense technical documentation       Test several sizes; preserve
                                      headings and code blocks where
                                      possible
  -----------------------------------------------------------------------

These are experimentation ranges, not fixed rules. Consider token
limits, document structure, embedding model behavior, and the size of
the context window you intend to send to the LLM.

### Important distinction

`splitDocuments(docs)` splits existing LangChain `Document` objects and
carries their metadata forward to the resulting chunks. Metadata such as
the source filename or PDF page number is useful for citations and
debugging retrieval.

------------------------------------------------------------------------

## 3. Create embeddings and store chunks

Install the relevant integrations if they are not already in the
project:

``` bash
npm install @langchain/openai @langchain/classic
```

Create an embedding model:

``` ts
import { OpenAIEmbeddings } from "@langchain/openai";

const embeddings = new OpenAIEmbeddings({
  model: "text-embedding-3-large",
});
```

Create an in-memory vector store and add the chunks:

``` ts
import { MemoryVectorStore } from "@langchain/classic/vectorstores/memory";

const vectorStore = new MemoryVectorStore(embeddings);

await vectorStore.addDocuments(allSplits);
```

`addDocuments()` uses the configured embedding model to embed the
document contents and stores the resulting vectors alongside the
documents. You do not need to manually call the embedding model for
every chunk in this flow.

The embedding model used at query time must be compatible with the one
used to index the documents. In practice, use the same embedding model
and configuration for both ingestion and retrieval.

### What is `MemoryVectorStore`?

`MemoryVectorStore` is useful for learning, local experiments, and small
demonstrations. Its data is held in process memory and is not a durable
production database. Restarting the process loses the indexed data.

For production, select a persistent vector database or a database with
vector-search support. Consider persistence, metadata filtering, access
control, backups, scaling, and operational cost.

------------------------------------------------------------------------

## 4. Similarity search

A basic search looks like this:

``` ts
const results = await vectorStore.similaritySearch(
  "When was Nike incorporated?",
  4,
);

console.log(results);
```

The second argument is the number of documents to return (`k`). Each
result is a LangChain `Document`, typically containing:

-   `pageContent`: the retrieved text.
-   `metadata`: information such as source file, page, or other loader
    metadata.

The vector store embeds the query and compares it with stored document
vectors. It returns chunks that are semantically close to the query.

**Important:** semantic similarity is not proof that a chunk contains
the answer. A retrieval result can be related to the question but still
lack the required fact. The LLM cannot reliably answer from evidence
that retrieval failed to supply.

------------------------------------------------------------------------

## 5. Configure retrieval with a retriever

A retriever provides a standard interface for fetching relevant
documents. It lets you configure the retrieval strategy rather than
always calling basic similarity search directly.

``` ts
const retriever = vectorStore.asRetriever({
  searchType: "mmr",
  searchKwargs: {
    fetchK: 10,
    k: 4,
  },
});

const retrievedDocs = await retriever.invoke(
  "When was Nike incorporated?",
);
```

Exact option names and supported values can vary by LangChain.js version
and vector-store integration. Check the installed type definitions and
documentation. In particular, do not assume every vector store supports
every search strategy.

### Common search strategies

#### Similarity search

Returns the chunks most similar to the query. This is a good baseline
when the most relevant passages are what matter.

#### MMR (Maximal Marginal Relevance)

MMR balances relevance with diversity. It attempts to avoid returning
several chunks that contain nearly identical information.

This can be useful when the top similarity results are repetitive. It
may return fewer highly similar passages in favor of a broader set of
relevant passages.

#### Similarity score threshold

Some vector stores and retriever configurations support a
score-threshold strategy. It filters results based on a configured
relevance threshold. Score meaning and direction (for example, distance
versus similarity) depend on the integration, so verify the
implementation before choosing a threshold.

### `k` and `fetchK`

-   **`k`**: the number of final documents to return.
-   **`fetchK`**: for MMR implementations, the number of candidate
    documents considered before selecting the final diverse set.

For example, an MMR search may consider 10 candidates (`fetchK: 10`) and
return 4 (`k: 4`). The exact configuration shape is integration/version
dependent.

Tune retrieval by inspecting the actual chunks returned for a test set
of questions. Do not choose `k`, `fetchK`, or a search strategy solely
because a tutorial uses a particular value.

------------------------------------------------------------------------

## 6. Two-step RAG with dynamic system-prompt middleware

In two-step RAG, retrieval happens before the LLM generates the answer:

1.  Read the user's question.
2.  Search the vector store.
3.  Extract and format the retrieved text.
4.  Add that text to the system instructions as context.
5.  Call the LLM to answer the question using that context.

A simplified example based on the lecture:

``` ts
import { createAgent, dynamicSystemPromptMiddleware } from "langchain";

const ragMiddleware = dynamicSystemPromptMiddleware(async (state) => {
  const lastMessage = state.messages.at(-1)?.content;
  const query = typeof lastMessage === "string" ? lastMessage : "";

  const retrievedDocs = await vectorStore.similaritySearch(query, 2);

  const docsContent = retrievedDocs
    .map((doc) => doc.pageContent)
    .join("\n\n");

  return `You are a helpful assistant.
Use the following context from the documents to answer the user's question.
If the context does not contain the answer, say that you cannot find it in the provided documents.

Context:
${docsContent}`;
});

const agent = createAgent({
  model: "gpt-4o",
  tools: [],
  middleware: [ragMiddleware],
});

const result = await agent.invoke({
  messages: [
    {
      role: "user",
      content: "When was Nike incorporated?",
    },
  ],
});

console.log(result);
```

### Why use middleware?

The middleware can retrieve context and dynamically construct the system
prompt before the model call. This keeps retrieval logic separate from
the agent's main setup.

### Important implementation notes

-   Use the **latest user message** as the retrieval query.
    `state.messages[0]` is only safe in a single-turn example; in a
    conversation it may refer to an earlier message.
-   Depending on the message type, content may not be a plain string.
    Multimodal messages can contain structured content blocks. Handle
    those explicitly if your application supports them.
-   Tell the model what to do when the retrieved context does not
    contain the answer. This reduces unsupported guesses.
-   Treat retrieved document text as untrusted input. It may contain
    instructions; the system prompt should tell the model to treat it as
    source material, not as instructions to follow.
-   For production, consider returning source metadata and page
    references along with the text so the answer can cite its evidence.

------------------------------------------------------------------------

## 7. Load multiple PDF files

The ingestion pipeline does not fundamentally change when there are
multiple PDFs. Load each file, combine the resulting `Document` objects,
and split the combined collection.

``` ts
import { PDFLoader } from "@langchain/community/document_loaders/fs/pdf";

const pdfPaths = [
  "./docs/nike-10k-2023.pdf",
  "./docs/nike-10k-2025.pdf",
  "./docs/nike-growth-story.pdf",
];

const allDocs = [];

for (const pdfPath of pdfPaths) {
  const loader = new PDFLoader(pdfPath);
  const docs = await loader.load();

  allDocs.push(...docs);
}

const allSplits = await textSplitter.splitDocuments(allDocs);
console.log("Total chunks:", allSplits.length);

await vectorStore.addDocuments(allSplits);
```

The spread operator (`...docs`) adds each loaded document to `allDocs`
as an individual array element.

For a large number of files, consider controlled concurrency, ingestion
batching, retries, and progress reporting rather than loading everything
at once.

### Why metadata matters with multiple files

Keep the source filename and page metadata. If a retrieved chunk
contains the answer, the application should be able to identify which
document and page it came from.

------------------------------------------------------------------------

## 8. Load Word documents (DOCX)

For `.docx` files, LangChain provides a DOCX loader through the
community package. Install its required dependency as described by the
current LangChain.js documentation:

``` bash
npm install mammoth
```

Example:

``` ts
import { DocxLoader } from "@langchain/community/document_loaders/fs/docx";

const loader = new DocxLoader("./docs/nike-growth-story.docx");
const docs = await loader.load();

const allSplits = await textSplitter.splitDocuments(docs);
await vectorStore.addDocuments(allSplits);
```

The downstream process is the same:

``` text
DOCX loader
  -> Document objects
  -> Text splitter
  -> Embeddings
  -> Vector store
  -> Retriever
  -> LLM
```

Only the document-loading step changes. The same general approach can be
applied to other formats using their corresponding loaders, such as CSV
or other supported document loaders.

------------------------------------------------------------------------

## 9. Two-step RAG vs. agentic RAG

### Two-step RAG

Retrieval is performed as a fixed step before generation.

``` text
User question
    |
    v
Retrieve relevant documents
    |
    v
Build context
    |
    v
LLM generates answer
```

Use this when most questions should always be answered using the
document collection. It is straightforward, predictable, and avoids
asking the model to decide whether retrieval is needed.

### Agentic RAG

Retrieval is exposed to the agent as a tool. The model decides when and
how to call it.

``` text
User question
    |
    v
Agent / LLM
    |
    +----> Retrieval tool ----> Vector store
    |                              |
    |<--------- Retrieved context-+
    |
    v
LLM generates answer
```

Use this when the agent has several tools and must decide whether
document retrieval is appropriate, possibly alongside other actions such
as weather lookup or other application tools.

The trade-off is that agentic RAG gives the model more flexibility, but
tool selection and additional model/tool calls can add latency and cost.
Two-step RAG is often preferable when retrieval is mandatory for every
query.

------------------------------------------------------------------------

## 10. Agentic RAG: expose retrieval as a tool

A simplified version of the lecture's tool pattern:

``` ts
import { tool } from "langchain";
import { z } from "zod";

const retrieve = tool(
  async ({ query }) => {
    const retrievedDocs = await vectorStore.similaritySearch(query, 2);

    return retrievedDocs
      .map((doc) => doc.pageContent)
      .join("\n\n");
  },
  {
    name: "retrieve",
    description: "Search the indexed company documents for relevant information.",
    schema: z.object({
      query: z.string(),
    }),
  },
);
```

Register the tool with the agent:

``` ts
const agent = createAgent({
  model: "gpt-4o",
  tools: [retrieve],
  systemPrompt:
    "Use the document retrieval tool when the user asks questions about the indexed company documents. Base document answers on retrieved evidence.",
});
```

The model can decide whether to call `retrieve` based on the user's
request and the tool description.

A tool description should explain: - What information the tool can
retrieve. - When the model should use it. - What its input means. - What
kind of output it returns.

Do not expose unrelated tools unless the application needs them.
Validate tool inputs and enforce authorization in the tool
implementation, not only in the prompt.

------------------------------------------------------------------------

## 11. Practical debugging and evaluation checklist

When an answer is incorrect or missing, debug the RAG pipeline in order.

### Ingestion

-   [ ] Did the loader read the expected files?
-   [ ] Does `docs.length` look reasonable?
-   [ ] Does `pageContent` contain the expected text?
-   [ ] Is metadata (filename, page number, etc.) preserved?
-   [ ] Did the splitter produce a reasonable number of chunks?
-   [ ] Are chunks cut at sensible boundaries?
-   [ ] Are chunk size and overlap appropriate for the content?

### Embeddings and vector storage

-   [ ] Is the embedding model configured correctly?
-   [ ] Was every intended chunk added to the vector store?
-   [ ] Is the same compatible embedding configuration used for query
    embedding?
-   [ ] Is the vector store persistent if the application requires data
    to survive restarts?

### Retrieval

-   [ ] Does the query retrieve the chunk containing the answer?
-   [ ] Are the returned chunks relevant, not merely keyword-adjacent?
-   [ ] Is `k` too small or unnecessarily large?
-   [ ] Would MMR reduce repetitive results?
-   [ ] Are metadata filters needed to restrict results by tenant,
    document, date, or access permissions?

### Generation

-   [ ] Is the retrieved context actually included in the model call?
-   [ ] Is the model instructed to answer only from the supplied
    evidence?
-   [ ] Does it clearly say when the answer is not present?
-   [ ] Can the answer be traced to a source document and page?

**Debug retrieval before changing the LLM prompt.** If the right
evidence is not retrieved, a stronger generation prompt cannot reliably
recover it.

------------------------------------------------------------------------

## 12. Corrections and cautions from the lecture examples

1.  **Chunk size is not token count.** The default recursive character
    splitter measures characters, unless a custom `lengthFunction` is
    supplied.
2.  **Overlap is between adjacent chunks.** It preserves boundary
    context; it does not mean the splitter scans an extra 200 characters
    in both directions as a separate search operation.
3.  **The vector store does not answer the question.** It returns
    relevant document chunks. The LLM generates the final answer from
    those chunks.
4.  **Similarity does not guarantee factual relevance.** Evaluate
    retrieval with real questions and known expected source passages.
5.  **Do not assume the first message is the current question.** Use the
    latest user message or explicitly identify the message to retrieve
    against.
6.  **`MemoryVectorStore` is not a production persistence layer.** Use a
    persistent store for a deployed application.
7.  **Verify retriever option shapes.** `searchType`, `searchKwargs`,
    `fetchK`, and `k` support can differ between LangChain.js versions
    and vector-store integrations.
8.  **Do not silently accept missing evidence.** Add an explicit
    fallback instruction and ideally a relevance threshold or validation
    step.
9.  **Preserve citations and metadata.** Return source names and page
    numbers with retrieved content when users need to verify answers.
10. **Avoid indexing duplicates.** Repeated documents or repeated
    ingestion runs can produce duplicate chunks and distort retrieval.

------------------------------------------------------------------------

## 13. Key takeaways

-   Load files into LangChain `Document` objects.
-   Use `RecursiveCharacterTextSplitter` to create manageable,
    overlapping chunks.
-   Convert chunks into vectors using an embedding model such as
    `text-embedding-3-large`.
-   Store the vectors and original document content in a vector store.
-   Retrieve relevant chunks using similarity search or a configured
    retriever.
-   Use the retrieved content as context for the LLM.
-   Choose two-step RAG for a fixed retrieval-first flow; choose agentic
    RAG when the model should decide whether to use retrieval as one of
    several tools.
-   Measure retrieval quality with representative questions instead of
    relying on one successful demo.

## References

-   [LangChain.js documentation](https://js.langchain.com/)
-   [LangChain text
    splitters](https://docs.langchain.com/oss/javascript/integrations/splitters)
-   [LangChain document
    loaders](https://docs.langchain.com/oss/javascript/integrations/document_loaders/)
-   [LangChain vector
    stores](https://docs.langchain.com/oss/javascript/integrations/vectorstores/)
-   [OpenAI embeddings
    guide](https://platform.openai.com/docs/guides/embeddings)
