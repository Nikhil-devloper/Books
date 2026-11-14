# Retrieval Augmented Generation (RAG) --- Study Notes

> **Course:** LangChain\
> **Topic:** Retrieval architecture, embeddings, semantic search,
> retrieval pipeline, and PDF loading\
> **Scope:** Notes based on the lecture transcript and diagrams.

------------------------------------------------------------------------

## 1. What is RAG?

**RAG stands for Retrieval Augmented Generation.**

RAG is an architecture in which an application retrieves relevant
information from external sources---such as PDFs, Word documents, or
company documents---and provides that information to an LLM as context
for generating an answer.

Instead of relying only on its pre-trained knowledge, the LLM receives
relevant external information at runtime.

### Why do we need RAG?

The lecture highlights two limitations of LLMs.

#### 1.1 Finite context window

-   An LLM can process only a limited amount of information in one
    request.
-   Imagine a company has 100 PDF documents containing policies and
    other information.
-   Sending all 100 PDFs to the LLM may exceed its context limit.
-   Processing large amounts of content can also increase token usage
    and cost.

#### 1.2 Training data is frozen at a point in time

-   A pre-trained LLM's knowledge comes from the data on which it was
    trained.
-   It may not know about information that has changed after its
    training cutoff.
-   For example, an insurance provider may update its policies every
    week.
-   The LLM may return outdated information if it does not have access
    to the latest policy details.

### How RAG helps

RAG retrieves only the information relevant to the user's question and
sends that information to the LLM along with the original query.

**Example:**

A bank has many documents containing:

-   Saving account interest rates
-   Insurance policies
-   Loan information
-   Company policies
-   Banking guidelines

A user asks:

> Which insurance policy provides the maximum coverage?

Instead of sending every document to the LLM:

1.  Search the document collection.
2.  Retrieve the most relevant information.
3.  Send the original question and retrieved information to the LLM.
4.  Let the LLM generate the answer using that context.

This reduces the amount of information sent to the LLM and allows an
application to use private or updated company information.

------------------------------------------------------------------------

## 2. RAG Architecture

RAG has three major stages:

  -----------------------------------------------------------------------
  Stage                               Meaning
  ----------------------------------- -----------------------------------
  **R --- Retrieval**                 Find relevant information from
                                      external sources.

  **A --- Augmentation**              Combine the original user query
                                      with the retrieved context.

  **G --- Generation**                Generate the final answer using an
                                      LLM.
  -----------------------------------------------------------------------

### Step-by-step architecture

#### Step 1: User query

The user asks a question.

Example:

> When was Nike incorporated?

#### Step 2: Generate query embedding

-   The user's query is passed to an embedding model.
-   The embedding model converts the query into a numerical vector.
-   This vector represents the semantic meaning of the query.

#### Step 3: Retrieve relevant context

-   The query embedding is used to search the Vector Database.
-   The database contains embeddings generated from previously stored
    documents.
-   Semantic similarity is used to identify relevant information.
-   The system retrieves the top relevant documents or chunks.

For example, out of 1,000 documents, the search may identify two
relevant documents.

#### Step 4: Augmentation

Two inputs are sent to the LLM:

1.  The original user query.
2.  The retrieved documents or context.

The LLM is instructed to answer using the retrieved context.

#### Step 5: Generation

-   The LLM reads the query and retrieved context.
-   It reasons over the provided information.
-   It generates the final response.

**Important:** The retriever finds relevant information; it does not
itself generate the final natural-language answer.

------------------------------------------------------------------------

## 3. What is an Embedding Model?

An **embedding model** converts text into a numerical representation
called a **vector embedding**.

Example input:

``` text
Nike was incorporated.
```

A simplified illustrative embedding might look like:

``` text
[0.234, 0.898, 0.451, 0.672, ...]
```

The actual embedding contains a number of dimensions determined by the
embedding model.

### How embeddings are used in RAG

There are two important operations.

#### 3.1 Document embedding

When preparing documents for storage:

1.  Read the document.
2.  Extract its text.
3.  Convert the text into embeddings.
4.  Store the embeddings in a Vector Database, along with the associated
    document information.

#### 3.2 Query embedding

When a user asks a question:

1.  Receive the original query.
2.  Convert the query into an embedding using the embedding model.
3.  Search the Vector Database using that embedding.
4.  Retrieve relevant document chunks.

The query and stored document chunks must use compatible embedding
representations for similarity search.

### Why do we need embeddings?

The main reason discussed in the lecture is **semantic search**.

  -----------------------------------------------------------------------
  Keyword / syntax search             Semantic search
  ----------------------------------- -----------------------------------
  Matches specific words or phrases.  Searches based on meaning.

  Focuses on matching keywords.       Focuses on semantic similarity.

  Can miss related information        Can find related information even
  expressed using different wording.  when the wording differs.
  -----------------------------------------------------------------------

**Example**

Stored document:

> An organization was established in 1964.

User query:

> When was the company founded?

A keyword search may not find an exact word match. Semantic search can
identify that both sentences discuss the company's establishment.

**Important:** Embeddings do not directly generate answers. They help
identify relevant information. The LLM generates the final answer.

------------------------------------------------------------------------

## 4. What is a Vector Database?

A **Vector Database** stores and searches numerical vector embeddings.

In RAG, it acts as the storage and retrieval layer for document
information.

### How it works

Suppose there are 1,000 company documents:

1.  Documents are converted into text.
2.  Text is split into smaller chunks.
3.  Chunks are converted into embeddings.
4.  Embeddings and associated document information are stored in the
    Vector Database.
5.  A user's query is converted into an embedding.
6.  The database finds the most relevant stored vectors.
7.  The corresponding document chunks are retrieved.

### Components and responsibilities

  -----------------------------------------------------------------------
  Component                           Responsibility
  ----------------------------------- -----------------------------------
  **Embedding Model**                 Converts text into numerical
                                      vectors.

  **Vector Database / Vector Store**  Stores and searches vector
                                      embeddings.

  **Retriever**                       Fetches relevant documents or
                                      chunks.

  **LLM**                             Uses the retrieved context to
                                      generate an answer.
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 5. Retrieval Pipeline

Before implementing RAG, documents must be prepared and stored in a
Vector Database. This process is called the **retrieval pipeline** or
document ingestion pipeline.

### Document ingestion flow

``` text
Sources
  |
  v
Document Loaders
  |
  v
Document Objects
  |
  v
Split into Chunks
  |
  v
Generate Embeddings
  |
  v
Vector Store
```

#### Step 1: Sources

Documents can come from different sources, for example:

-   Google Drive
-   Slack
-   Notion
-   PDF files
-   Word documents
-   Other supported content sources

#### Step 2: Document loaders

Document loaders load and extract content from different sources and
file formats.

#### Step 3: Document objects

The extracted content is represented as document objects that can be
processed by the application.

#### Step 4: Split into chunks

Large documents are divided into smaller pieces. The lecture introduces
this step and explains that chunking will be covered in the next
lecture.

#### Step 5: Generate embeddings

The text chunks are converted into numerical vector embeddings.

#### Step 6: Store in a Vector Store

The embeddings and associated document information are stored so that
they can be searched later.

### Query-time retrieval flow

After documents have been ingested:

``` text
User Query
    |
    v
Query Embedding
    |
    v
Vector Store
    |
    v
Retriever
    |
    v
Relevant Document Chunks
    |
    v
LLM receives Query + Context
    |
    v
Answer
```

**Remember the distinction:**

-   **Document ingestion** prepares and stores documents before a user
    asks a question.
-   **Query-time retrieval** searches the stored information when a user
    asks a question.

------------------------------------------------------------------------

## 6. Document Loaders in LangChain

Document Loaders load data from different sources and convert it into
document objects that can be processed by LangChain.

Different file formats may require different loaders.

Examples mentioned in the lecture:

  Source / Format   Loader
  ----------------- ----------------------
  PDF               PDF Loader
  CSV               CSV Loader
  JSON              JSON Loader
  Word documents    Word document loader
  Text              Text loader
  Markdown          Markdown loader

The loader is selected based on the source format.

### What does a Document object contain?

In the practical lecture, a 107-page PDF is loaded. The PDF Loader
generates 107 document objects in this example.

Each document object contains two important properties:

1.  **`pageContent`** --- the text extracted from the page.
2.  **`metadata`** --- additional information about the document, such
    as page and source information.

Conceptual example:

``` typescript
{
  pageContent: "Extracted PDF content...",
  metadata: {
    // Document metadata
  }
}
```

------------------------------------------------------------------------

## 7. Practical Implementation --- PDF Loader with TypeScript

The practical implementation starts with a PDF document.

### Step 1: Create a RAG folder

Keep RAG-related implementation separate from the existing LangChain
code.

Example project structure:

``` text
project/
├── rag/
│   ├── .env
│   └── agent1.ts
├── docs/
│   └── sample.pdf
└── package.json
```

The instructor also recommends having an environment file inside the RAG
folder.

### Step 2: Install dependencies

The lecture uses LangChain Community and PDF parsing dependencies.

``` bash
npm install @langchain/community pdf-parse
```

The instructor encounters dependency-resolution issues during
installation and adjusts package versions. Use versions compatible with
the LangChain setup in your project.

### Step 3: Import PDFLoader

``` typescript
import { PDFLoader } from "@langchain/community/document_loaders/fs/pdf";
```

### Step 4: Initialize PDFLoader

``` typescript
const loader = new PDFLoader("./docs/sample.pdf");
```

-   `PDFLoader` is the document loader class.
-   The PDF file path is passed as an argument.
-   The loader reads and parses the PDF.

### Step 5: Load the PDF

``` typescript
const docs = await loader.load();
```

The `load()` method is asynchronous, so use `await`.

The returned `docs` variable contains the extracted document objects.

### Step 6: Inspect the loaded documents

Print the document objects:

``` typescript
console.log(docs);
```

Check how many document objects were generated:

``` typescript
console.log(docs.length);
```

Access the first document's extracted content:

``` typescript
console.log(docs[0].pageContent);
```

### Expected result from the lecture

For the instructor's 107-page PDF:

``` text
docs.length = 107
```

In this example, the PDF Loader generates one document object per page.

Each document object contains:

-   `pageContent`
-   `metadata`

**Important:** At this stage, the PDF content has only been extracted.
It has not yet been split into smaller chunks or converted into
embeddings.

------------------------------------------------------------------------

## 8. What Comes Next?

The practical implementation has reached the document-loading stage.

The remaining stages are:

  -----------------------------------------------------------------------
  Step                    Concept                 Purpose
  ----------------------- ----------------------- -----------------------
  1                       Document Loader         Load the PDF and
                                                  extract content.

  2                       Text Splitter           Divide documents into
                                                  smaller chunks.

  3                       Embedding Model         Convert chunks into
                                                  numerical vectors.

  4                       Vector Store            Store embeddings and
                                                  associated document
                                                  information.

  5                       Retriever               Retrieve relevant
                                                  chunks.

  6                       LLM                     Generate answers using
                                                  the retrieved context.
  -----------------------------------------------------------------------

The next lecture will cover **text splitting and chunking**, including
why documents are divided into smaller pieces.

------------------------------------------------------------------------

## 9. Quick Revision --- Important Terms

  Term              Meaning
  ----------------- ----------------------------------------------------
  RAG               Retrieval Augmented Generation
  Retrieval         Finding relevant information from external sources
  Augmentation      Combining the user query with retrieved context
  Generation        Producing the final answer using an LLM
  Embedding         Numerical representation of text
  Embedding Model   Converts text into vectors
  Vector Database   Stores and searches embeddings
  Semantic Search   Searching based on meaning
  Document Loader   Loads and parses external documents
  Document Object   Contains page content and metadata
  Chunking          Splitting large documents into smaller pieces
  Retriever         Fetches relevant document chunks

------------------------------------------------------------------------

## 10. Key Takeaways

1.  RAG helps an LLM answer questions using relevant external
    information.
2.  RAG has three stages: Retrieval, Augmentation, and Generation.
3.  An embedding model converts text into numerical vectors.
4.  Semantic search finds information based on meaning rather than only
    exact keyword matches.
5.  A Vector Database stores and searches embeddings.
6.  A retriever fetches relevant document chunks from the Vector Store.
7.  The LLM receives the original query and retrieved context to
    generate the answer.
8.  Document loaders convert source files into document objects.
9.  In the practical example, a 107-page PDF produces 107 document
    objects.
10. Document loading is only the first step; chunking, embedding,
    storage, and retrieval are still required.

### RAG flow to remember

``` text
Documents
   |
   v
Document Loader
   |
   v
Document Objects
   |
   v
Chunks
   |
   v
Embeddings
   |
   v
Vector Database
   |
   v
Retriever
   |
   v
LLM (Original Query + Retrieved Context)
   |
   v
Answer
```

------------------------------------------------------------------------

## 11. Self-Assessment Questions

Use these questions to check your understanding.

1.  What does RAG stand for?
2.  What are the two major limitations of LLMs discussed in the lecture?
3.  Why do we need an embedding model?
4.  What is the difference between keyword search and semantic search?
5.  What is a Vector Database responsible for?
6.  What are the three stages of RAG?
7.  What is the difference between document ingestion and query-time
    retrieval?
8.  What is the responsibility of a Document Loader?
9.  What are the two important properties of a LangChain Document
    object?
10. Why does the retriever return relevant context instead of generating
    the final answer?
11. What does `await loader.load()` do?
12. In the lecture's example, why did a 107-page PDF produce 107
    document objects?

------------------------------------------------------------------------

## 12. Interview Questions

### Q1. What is RAG?

RAG stands for Retrieval Augmented Generation. It is an architecture in
which relevant information is retrieved from external sources and
supplied to an LLM as context to generate an answer.

### Q2. Why is RAG required when we already have powerful LLMs?

LLMs have finite context windows and their pre-trained knowledge is
frozen at a point in time. RAG allows an application to retrieve
relevant external information at runtime instead of sending an entire
document collection to the LLM or relying only on its training data.

### Q3. What is an embedding?

An embedding is a numerical vector representation of text. It captures
semantic information and can be used to find text with similar meaning.

### Q4. What is semantic search?

Semantic search finds information based on the meaning of a query rather
than relying only on exact keyword matches.

### Q5. What is the difference between an embedding model and a Vector Database?

An embedding model converts text into numerical vectors. A Vector
Database stores and searches those vectors.

### Q6. What is the role of a retriever in RAG?

A retriever searches the Vector Store and returns document chunks
relevant to the user's query. The LLM then uses those chunks to generate
the answer.

### Q7. What is a Document Loader in LangChain?

A Document Loader reads data from a source, such as a PDF, and converts
it into document objects containing extracted content and metadata.

### Q8. What is the difference between `pageContent` and `metadata`?

`pageContent` contains the extracted text. `metadata` contains
additional information about the document, such as its source or page
information.

------------------------------------------------------------------------

**End of notes --- Next topic: Text Splitters and Chunking.**
