---
title: "Empowering Developers with a Gen AI Technical Support Agent: A Case Study of Canva Apps SDK"
slug: "gen-ai-technical-support-agent"
date: 2023-11-26T00:00:00+11:00
draft: false
description: "A prototype RAG support agent that helps developers navigate the Canva Apps SDK and generate working examples."
tags: ["generative AI","RAG","developer experience","Canva"]
canonicalURL: "https://dxiaochuan.medium.com/empowering-developers-with-a-gen-ai-technical-support-agent-a-case-study-of-canva-sdk-d99496077a07"
---

*Originally published on [Medium](https://dxiaochuan.medium.com/empowering-developers-with-a-gen-ai-technical-support-agent-a-case-study-of-canva-sdk-d99496077a07).*

TL;DL: This blog discusses a prototype solution for aiding developers in using new tools and frameworks efficiently. It highlights the use of large language models (LLMs) and the retrieval augmented generation (RAG) method to create a self-service framework, enabling developers to resolve technical queries independently. We use the Canva Apps SDK as a case study to illustrate this process. It involves building a knowledge base from Canva Apps SDK documents, utilizing AI-driven chatbots for interactive support, and demonstrating the process through a practical example of deploying a Canva App. The blog also outlines improvement opportunities for productionalizing this prototype, emphasizing the need for a robust web crawler, leveraging a search engine with vector database support like OpenSearch, and fine-tuning AI models for more tailored content generation.

## Table of contents

- Introduction

- Canva, Canva App and Canva Apps SDK

- Building a knowledge base

- Retrieving Relevant Documents

- Using an LLM to generate responses for developers’ query

- Integrating a Chatbot UI

- Utilizing the Canva Apps SDK Chatbot

- Conclusion

## 1. Introduction

Supporting developers navigating the complexities of new tools and frameworks is crucial for developer-centric companies and open-source communities. Developers, particularly those new to a particular framework or tool, depend extensively on documentation and tutorials to build their expertise. However, when they face challenges, access to expert advice becomes essential to overcome hurdles and advance their work.

Conversely, the obligation of maintaining developer support teams to address technical inquiries represents a significant commitment for the entities offering these technologies. This responsibility requires substantial time and deep domain knowledge to resolve developers' queries effectively.

Imagine a world where developers can solve their technical challenges instantly with AI-powered support, and at the same time, entities can allocate human expert resources to more innovative work. A self-service framework that allows developers to find answers to their technical queries independently would be invaluable. Such a solution would significantly alleviate common pain points for both developers and the organizations supporting them.

Large Language Models (LLMs) have demonstrated their utility in crafting interactive interfaces, such as chatbots, to facilitate personalized and professional dialogues between customers and organizations. The [Retrieval Augmented Generation (RAG)]({{< ref "/posts/optimizing-retrieval-for-rag-applications" >}}) method is one way to integrate context in providing personalized, business-aware communication. By building a comprehensive knowledge base, organizations can leverage it to respond to customer inquiries. This method significantly reduces the load on an organization’s developer support team, freeing them from constantly responding to developers’ queries around the clock. It also ensures a consistent and smooth experience for developers seeking information about a specific framework or tool.

This blog post will explore a specific example of employing this strategy to accelerate application development using the Canva Apps SDK and demonstrate how to deploy a Canva App in under 30 minutes.

## 2. Canva, Canva App and Canva Apps SDK

[Canva](https://www.canva.com/about/) stands out as a comprehensive online design and publishing platform, enabling people worldwide to create and publish diverse designs easily. Within Canva, a unique element known as [Canva Apps](https://www.canva.com/developers/) adds to this versatility. Fundamentally, Canva Apps are JavaScript files operating within an iframe. These files are adept at generating a user interface within Canva’s environment, allowing interaction with various APIs to engage with the design process. [Canva Apps SDK](https://www.canva.dev/docs/apps/) includes Canva’s purpose-built APIs, a range of tools, and a supportive community to build your Canva App.

Despite the advancements in the Canva Apps SDK, which was designed to streamline the creation of Canva Apps, developers often face challenges. This includes a learning curve in mastering the implementation of specific features and successfully deploying these features as functional apps within Canva. Moreover, developers require support when developing applications using this framework. While the [Apps SDK documentation](https://www.canva.dev/docs/apps/) offers detailed guidance and FAQs for adopting the SDK, navigating through this content can be time-consuming. Additionally, the provided examples may not always align with developers’ specific needs.

We aim to establish a support service that promptly delivers technical answers to developers’ queries. This service will also provide customized Canva App examples that cater to the specific requirements of developers.

In the following section, we are going to implement a Gen AI support agent step-by-step.

## 3. Building a knowledge base

The first step involves downloading Canva Apps SDK documents and constructing a knowledge base using this information. This can be achieved by executing the following code in a notebook:

```
!pip install gradio python-dotenv openai llama-index rank_bm25 typing-extensions -q
```

```
from pathlib import Path

DATA_ROOT = Path('./data')
RAW_HTML_ROOT = DATA_ROOT / 'web_pages'
```

```
%%bash -s "$RAW_HTML_ROOT"
export RAW_HTML_ROOT=$1
wget -e robots=off --recursive --no-clobber --page-requisites \
  --html-extension --convert-links --restrict-file-names=windows \
  --domains www.canva.dev --no-parent --accept=html \
  -P $RAW_HTML_ROOT https://www.canva.dev/docs/apps/ || true
```

Then, we can read the HTML files we downloaded and extract text from the web pages:

```
import html2text

# Initialize html2text converter
converter = html2text.HTML2Text()
converter.ignore_links = True

# Function to read an HTML file and convert it to text
def convert_html_to_text(filename):
    with open(filename, 'r', encoding='utf-8') as file:
        html_content = file.read()
        return converter.handle(html_content)

# Convert each HTML file to text
text_list = [convert_html_to_text(filename) for filename in html_files]
```

If you print out one document page, you can see something like the following:

Next, we will employ the [llama_index](https://www.llamaindex.ai/) with[OpenAI’s Embedding API](https://platform.openai.com/docs/guides/embeddings) to create a local vector database, storing semantic information in [embeddings](https://developers.google.com/machine-learning/crash-course/embeddings/video-lecture). This vector database enables natural language processing and equates to a semantic knowledge base. Thanks to the llama_index implementation, building this database locally requires just a few lines of code.

```
import os
from llama_index import VectorStoreIndex, SimpleDirectoryReader
from llama_index import VectorStoreIndex, SimpleDirectoryReader, StorageContext, load_index_from_storage
from llama_index import download_loader

StringIterableReader = download_loader("StringIterableReader")

loader = StringIterableReader()

documents = loader.load_data(texts=text_list)
# attach page URL to documents
urls = [str(file_dir)[len(str(RAW_HTML_ROOT))+1:] for file_dir in html_files]
for url, doc in zip(urls, documents):
    doc.metadata = {"url": url}

storage_path = Path('./docs_storage')
# check if storage already exists
if (not os.path.exists(storage_path)):
    # load the documents and create the index
    index = VectorStoreIndex.from_documents(documents, show_progress=True)
    # store it for later
    index.storage_context.persist()
else:
    # load the existing index
    storage_context = StorageContext.from_defaults(persist_dir=storage_path)
    index = load_index_from_storage(storage_context)
```

Note: Ensure your `OPENAI_API_KEY` is set in the environment variables before running this code, as it calls the OpenAI API to encode text into embeddings. You can also set the key by running the following code:

```
import openai

openai.api_key = "<you-openai-api-key>"
```

## 4. Retrieving Relevant Documents

Providing sufficient context for our LLM model when answering developers’ queries is crucial. We can use llama_index retrieval functions for this purpose. We’ve constructed a semantic knowledge base that enables querying documents using natural language, supplemented by a text index to support lexical searches. The [BM25](https://en.wikipedia.org/wiki/Okapi_BM25) algorithm, renowned for full-text lexical searches, is employed here to implement the lexical searches.

```
from llama_index import (
    SimpleDirectoryReader,
    ServiceContext,
    StorageContext,
    VectorStoreIndex,
)
from llama_index.llms import OpenAI
from llama_index.retrievers import BM25Retriever
from llama_index.tools import RetrieverTool
from llama_index.indices.vector_store.retrievers.retriever import (
    VectorIndexRetriever,
)
from llama_index.retrievers import RouterRetriever

# initialize service context (set chunk size)
llm = OpenAI(model="gpt-4-1106-preview") # gpt-3.5-turbo
service_context = ServiceContext.from_defaults(chunk_size=1024, llm=llm)
nodes = service_context.node_parser.get_nodes_from_documents(documents)

vector_retriever = VectorIndexRetriever(index)
bm25_retriever = BM25Retriever.from_defaults(nodes=nodes, similarity_top_k=2)

retriever_tools = [
    RetrieverTool.from_defaults(
        retriever=vector_retriever,
        description="Useful in most cases",
    ),
    RetrieverTool.from_defaults(
        retriever=bm25_retriever,
        description="Useful if searching about specific information and keywords",
    ),
]

retriever = RouterRetriever.from_defaults(
    retriever_tools=retriever_tools,
    service_context=service_context,
    select_multi=True,
)
```

The llama_index’s RouterRetriever function utilizes OpenAI’s [tool selection feature](https://platform.openai.com/docs/assistants/tools), allowing an LLM model to choose between semantic or lexical search based on the query’s content. We describe each tool’s usage by setting the `description` tool shown above. The one LLM calling later will play the decision maker in choosing the appropriate retriever by analyzing the content of the query on the fly.

If you put a query like `How to handle errors in Canva Apps SDK?` You may end up with the following documents:

## 5. Using an LLM to generate responses for developers’ query

Let’s leverage the retriever we created with a query engine that can answer users’ queries.

```
from llama_index.indices.query.query_transform.base import StepDecomposeQueryTransform
from llama_index.indices.query.query_transform import HyDEQueryTransform
from llama_index.query_engine.multistep_query_engine import MultiStepQueryEngine
from llama_index import LLMPredictor
from llama_index.prompts.base import PromptTemplate
from llama_index.prompts.prompt_type import PromptType
from llama_index.schema import Node
from llama_index.query_engine import CitationQueryEngine

HYDE_TMPL = (
    "You are transform a human input query to search query of a Canva Apps SDK document index. Please write the query to answer the question.\n"
    "Try to only include details if they can help find relevant documents.\n"
    "\n"
    "\n"
    "{context_str}\n"
    "\n"
    "\n"
    'New Query:"""\n'
)

hyde_prompt = PromptTemplate(HYDE_TMPL, prompt_type=PromptType.SUMMARY)
# query rewrite transfomer
query_transform = HyDEQueryTransform(
    hyde_prompt=hyde_prompt
)

citation_query_engine = CitationQueryEngine(
    retriever=retriever,
)

query_engine = MultiStepQueryEngine(
    query_engine=citation_query_engine,
    query_transform=query_transform,
    index_summary="Used to answer questions about Canva Apps SDK documents",
)
```

We used the same query when we performed retrieval to generate a response. As you can see, in addition to the answer to the query, we also get the source URL, which developers can refer to. That’s great! Let’s put anything together and see some concrete examples.

## 6. Integrating a Chatbot UI

To surface the QA function, we can construct a simple Chatbot interface using [gradio](https://www.gradio.app/guides/creating-a-custom-chatbot-with-blocks):

```
import gradio as gr
import random
import time

with gr.Blocks() as demo:
    chatbot = gr.Chatbot()
    msg = gr.Textbox()
    clear = gr.ClearButton([msg, chatbot])

    def respond(message, chat_history):
        response = query_engine.query(message)
        links = [
            f"[Source {idx+1}]({source_link})"
            for idx, source_link in enumerate(list(set([node.metadata['url'] for node in response.source_nodes if 'url' in node.metadata])))
        ]
        bot_message = f"{response.response} \n\nSources can be found in : {', '.join(links)}"
        chat_history.append((message, bot_message))
        return "", chat_history

    msg.submit(respond, [msg, chatbot], [msg, chatbot])

demo.queue().launch(height=1500, debug=False)
```

As you can see, in addition to the response generated by the Gen AI agent, you also have access to the documents which helped the agent figure out the

## 7. Utilizing the Canva Apps SDK Gen AI Agent

Now, let’s apply our chatbot to a practical scenario. We’ll instruct the chatbot to build a Canva App with a drag-and-drop function. The prompt we use is `write an App example in React and typescript using Canva Apps SDK. In the app, there is a text 'Canva App'. Uses can drag and drop the text to the design` .

The agent provides the following results.

```
import React from "react";
import { DraggableText } from "components/draggable_text";

export function App() {
  return (
    <DraggableText>
      Canva App
    </DraggableText>
  );
}
```

Now, we can follow [Canva Developer Quick Start](https://www.canva.dev/docs/apps/quick-start/) to deploy the App. The only thing we need to change is to copy & paste the above TypeScirpt code to `src/app.tsx` , and then run `npm run start` .

After running the App, you can drag and drop your text from the App UI to the design.

Now, let’s try another example: we can copy & paste one question from the [Canva Developer Community](https://community.canva.dev/login). As you see above, the Gen AI agent helps us find the answer to the query.

## 8. Conclusion

This blog has journeyed through creating a Gen AI technical support agent, using the Canva Apps SDK as a practical example. Our journey began with acquiring and organizing Canva Apps SDK documentation into a knowledge base. We then developed a hybrid retrieval system to search relevant documents from the knowledge base. We built a simple Chatbot interface and integrated response generation with the retriever to provide an interactive user interface. Finally, we used a Canva App example to validate that the agent can provide correct answers and code examples.

In the blog, we use Python code to implement the entire flow. If you only want to quickly build a PoC without caring too much about performance tuning and customization, [GPT Crawler](https://www.builder.io/blog/custom-gpt) is a popular option for building a quick demo.

While we focused on the “happy path” of solution performance, a broader evaluation is essential for a holistic assessment of its utility to developers.

The current implementation can’t fully address a few edge cases. For example, if you ask `write an App example in React and typescript using Canva Apps SDK. In the app, there is an 'Add' button, when clicking the 'Add' button, the app adds a shape element to the user's design. The path of the shape element is a circle.` You may find that the Gen AI agent tries to follow the Image Element example to adapt to the Shape Element (not mentioned in the document directly). But if the agent has a chance to review another document referred by the retrieved document, the agent may have more context to create a correct answer. Using [llama-index Metadata](https://docs.llamaindex.ai/en/stable/examples/metadata_extraction/MetadataExtractionSEC.html) can help you build a more smart retrieval system.

You need to construct a comprehensive evaluation component to evaluate the solution’s performance across common query patterns. [Building RAG-based LLM Applications for Production](https://www.anyscale.com/blog/a-comprehensive-guide-for-building-rag-based-llm-applications-part-1) goes through a step-by-step process of RAG system evaluation. [Patterns for Building LLM-based Systems & Products](https://eugeneyan.com/writing/llm-patterns/) also cover several useful methods to evaluate RAG systems.

As we consider the next steps toward production, several enhancements emerge as critical. These include the development of more sophisticated web crawlers for ongoing knowledge ingestion, the application of advanced search engines like [OpenSearch](https://opensearch.org/blog/hybrid-search/) for scalable and efficient indexing, and fine-tuning pre-built models to align with specific content requirements. In particular, engaging with the [Canva Developer Community](https://community.canva.dev/login) could provide invaluable insights for refining our QA performance.

I hope this blog sheds light on how an RAG solution can help you improve developers' experience by providing instantaneous technical support. Happy coding!
