---
title: "Optimizing Retrieval for RAG Applications: Enhancing Contextual Knowledge in LLMs"
slug: "optimizing-retrieval-for-rag-applications"
date: 2023-08-15T00:00:00+10:00
draft: false
description: "A practical guide to building stronger RAG retrievers with sparse, dense, structured, and agent-assisted retrieval."
tags: ["RAG","LLM","information retrieval","generative AI"]
canonicalURL: "https://dxiaochuan.medium.com/optimizing-retrieval-for-rag-applications-enhancing-contextual-knowledge-in-llms-79ebcafe5f6e"
---

*Originally published on [Medium](https://dxiaochuan.medium.com/optimizing-retrieval-for-rag-applications-enhancing-contextual-knowledge-in-llms-79ebcafe5f6e).*

A Closer Look at RAG Retrievers

## TL;DL

- Enhancing the RAG retriever is a strategic move toward developing superior LLM applications.

- Merging techniques like Sparse Retrieval and Dense Retrieval from diverse data sources aids in creating more effective retrievers.

- LLMs can be leveraged to construct various components of the RAG retriever, utilizing tools like the Program-Aided language model (PAL) and SQL Agents.

- Adopting an approach of starting small and iterating fast is a reasonable strategy for building RAG applications.

## Introduction

We’re increasingly seeing chatbots and digital agents take over tasks like [helping customers order meals](https://www.wsj.com/articles/can-ai-replace-humans-we-went-to-the-fast-food-drive-through-to-find-out-193c03e9?utm_campaign=The+Batch&utm_medium=email&_hsmi=269671984&_hsenc=p2ANqtz--m0BRGG1L7fsqCrNy99ElhCLqaObRycHNuDAMI7EmAMNmuThN-fD-dzbTyIjD-sx4wuZfw9EqjJfcYYcu23hNG1AZG2g&utm_content=269661055&utm_source=hs_email) or [helping customers shop online](https://www.nytimes.com/2023/06/16/technology/how-to-use-ai-as-a-shopping-assistant.html). The reason for this? They make the process faster, simpler and even help businesses boost their sales through cross-selling. Large Language Models (LLMs) contribute to this advancement. These LLMs have been trained so well that they can not only take orders accurately, but they’re also experts at suggesting other items the customers might like.

But here’s the thing: LLMs are like blank slates about your specific business data. They aren’t aware of the items on your menu, bestsellers, stock levels, or what your customers usually order. So, we have to teach them about these by augmenting their ‘knowledge’ with your unique data. This way, they can communicate effectively with your customers.

Here comes into play a method we call Retrieval Augmented Generation (RAG). This strategy helps us pull the strings, ensuring the chatbots respond in a more specific and relevant way. Imagine having a conversation where someone gives you extra helpful information on the side that allows you to make more informed statements. That’s the principle behind RAG.

RAG is a two-step model: it has the retriever, which is like a librarian that goes and fetches the relevant information you need (domain knowledge), and then we have the generator, our specialist response writer who uses this knowledge to create a well-informed, clear, and helpful response.

For example, if a customer asks, ‘How do I zoom in when taking a photo on my phone?’, the retriever scurries through manuals, customer reviews, product descriptions, and more to find relevant info. Then the generator uses its inbuilt smarts combined with that info to guide your customer step by step.

In this blog, we’ll zoom in on the role of the retriever and explore ways to optimize its actions to increase the overall effectiveness of our RAG applications.

## Why do we need RAG?

LLMs can store factual knowledge in their parameters and achieve state-of-the-art natural language processing (NLP) tasks like question answering and summarization with relative ease. For example, you can use LLMs to build [a support chatbot](https://www.ultimate.ai/blog/ultimate-life/meet-ultimategpt-the-llm-powered-bot-to-revolutionize-your-support) to answer customer questions.

However, [LLMs’ ability to precisely access and apply knowledge still needs improvement](https://arxiv.org/abs/2005.11401), resulting in some miscommunication (or ‘hallucinations’), especially in knowledge-intensive tasks. Also, LLMs depend on their training data for knowledge acquisition. Hence, they are not privy to real-time or updated information. Moreover, since most LLMs were pre-trained on internet corpora, they require extra context or ‘domain knowledge’ when dealing with specific scenarios or industries.

Two popular solutions have emerged to combat these challenges:

One is to fine-tune LLMs with specific data. While effective, this method requires considerable data collection and model training resources and asks for extra investment in training infrastructure. It’s also worth noting that fine-tuning generally [enhances style alignment rather than ensuring the accuracy of facts](https://www.anyscale.com/blog/fine-tuning-is-for-form-not-facts), making it only a partial solution.

The second strategy includes incorporating specific knowledge directly into the generator’s prompt. While this method helps to compensate for the limitations, there’s a catch — LLMs often struggle with long prompts, forcing us to become selective about the information we retain in the prompts.

This is where the concept of RAG comes into play. RAG merges information retrieval processes with response generation, helping bridge the gap. RAG starts with searching informative [passage](https://paperswithcode.com/task/passage-retrieval)s from a data store, then constructs [prompts](https://haystack.deepset.ai/blog/beginners-guide-to-llm-prompting#:~:text=A%20prompt%20is%20an%20instruction,and%20has%20the%20right%20length.) using the fetched information, and finally use LLMs to generate a human-friendly response.

The efficiency of the RAG method depends not solely on LLMs’ capability but also on the quality and variety of information that can be retrieved and used. Modern LLMs like [GPT-4](https://openai.com/gpt-4), [LLama V2](https://ai.meta.com/llama/), and [Claude 2](https://www.anthropic.com/index/claude-2) are great at summarising content and producing human-friendly responses, making the retrievers the differentiator of premium DAG applications.

## How do we build a good RAG retriever?

Creating an efficient RAG retriever involves extracting pertinent information from chosen data stores which then aid in fostering responses grounded on the retrieved data.

The initial step to constructing a RAG retriever is understanding what information best complements the downstream generator’s abilities. Working backward from the use cases is a good starting point.

Let’s bring this concept into focus with a detailed example.

Imagine ‘Buy More From Us (BMFU),’ an e-commerce entity specializing in electric devices. BMFU’s machine learning team is exploring the possibility of a virtual shopping assistant to elevate the customer shopping experience. Tom, an engaged platform user, and potential loyal customer, is their primary focus.

For their initial development stage, BMFU aims to support several specific features in their assistant:

1. **Product feature-based query.** For instance, Tom might ask, “I have a budget of $200. I need a keyboard with colorful lighting for PC gaming. Do you have any recommendations?”
2. **How-to Q&A.** A question like, “I lost the ‘Q’ key from the keyboard I purchased last week. How can I get a replacement?” fits into this category.
3. **Interactive Q&A.** An example could be Tom on BMFU’s keyboard webpage asking, “Can this keyboard run on rechargeable batteries?”

With these requirements in mind, let’s discuss creating an effective RAG retriever.

### 1. Scoping the Available Data

We need product pricing and description information to answer a product feature-based query (# 1).To handle how-to Q&As, we require access to product manuals(# 2).

Online systems often store pricing information in a transactional store. So our assistant should be able to talk with SQL DB. In contrast, product descriptions and product manuals, generally in their rich text format, are typically stored in search engines to support full-text search.

We should incorporate multi-data sources during retrieval. For SQL DB, you can build a custom function to read data from DB. You can also leverage LLMs as a [SQL Agent](https://python.langchain.com/docs/integrations/toolkits/sql_database) to help structure SQL to query the data. For example, you use the following prompt to ask an LLM to generate an SQL query for you to query your data store:

```
# Use the the schema links and Intermediate_representation to generate the SQL queries for each of the questions.
Table advisor, columns = [*,s_ID,i_ID]
Table classroom, columns = [*,building,room_number,capacity]
Table course, columns = [*,course_id,title,dept_name,credits]
Table department, columns = [*,dept_name,building,budget]
Table instructor, columns = [*,ID,name,dept_name,salary]
Table prereq, columns = [*,course_id,prereq_id]
Table section, columns = [*,course_id,sec_id,semester,year,building,room_number,time_slot_id]
Table student, columns = [*,ID,name,dept_name,tot_cred]
Table takes, columns = [*,ID,course_id,sec_id,semester,year,grade]
Table teaches, columns = [*,ID,course_id,sec_id,semester,year]
Table time_slot, columns = [*,time_slot_id,day,start_hr,start_min,end_hr,end_min]
Foreign_keys = [course.dept_name = department.dept_name,instructor.dept_name = department.dept_name,section.building = classroom.building,section.room_number = classroom.room_number,section.course_id = course.course_id,teaches.ID = instructor.ID,teaches.course_id = section.course_id,teaches.sec_id = section.sec_id,teaches.semester = section.semester,teaches.year = section.year,student.dept_name = department.dept_name,takes.ID = student.ID,takes.course_id = section.course_id,takes.sec_id = section.sec_id,takes.semester = section.semester,takes.year = section.year,advisor.s_ID = student.ID,advisor.i_ID = instructor.ID,prereq.prereq_id = course.course_id,prereq.course_id = course.course_id]
Q: "Find the buildings which have rooms with capacity more than 50."
A: Let’s think step by step. In the question "Find the buildings which have rooms with capacity more than 50.", we are asked:
"the buildings which have rooms" so we need column = [classroom.capacity]
"rooms with capacity" so we need column = [classroom.building]
Based on the columns and tables, we need these Foreign_keys = [].
Based on the tables, columns, and Foreign_keys, The set of possible cell values are = [50]. So the Schema_links are:

Q: "Find the buildings which have the most number of classroom." # this is your query
A: Let’s think step by step. <LLM-completion>
```

More information on this topic can be found in [this paper](https://arxiv.org/pdf/2305.15038.pdf) and [LlamaIndex’s document.](https://gpt-index.readthedocs.io/en/v0.6.16/guides/tutorials/sql_guide.html)

In the following section, we will cover document retrieval approaches for the search engine.

### 2. Improve the possibility of finding a relevant context in a search engine

We need to build full-text search capacity to support # 1 and # 2.

Searching unstructured data is a long-existing requirement for information retrieval(IR) systems. Sparse retrieval (SR) and Dense retrieval (DR) are commonly used to support keyword-based search (lexical search) and semantic search.

SR projects the document to a sparse vector — as the name suggests — which typically aligns with the vocabulary of the document’s language. Some standard algorithms like TF-IDF or BM25 are SR methods. They are well-used in search engines like [ElasticSearch](https://www.elastic.co/blog/practical-bm25-part-2-the-bm25-algorithm-and-its-variables) and [Apache Solr](https://solr.apache.org/guide/7_0/learning-to-rank.html).

SR methods have advantages like low latency and explainability. But the sparse representation of text based on tokens only partially reflects each term’s semantics in the context of the whole text. Despite the wide usage, [these algorithms are handcrafted and therefore cannot be optimized for a specific task](https://openreview.net/pdf?id=rkg-mA4FDr).

More discussion on the considerations of using SR can be found in Appendix.

DR usually encodes queries and documents using single-vector representations (embeddings). Using embedding-based retrieval can leverage rich semantic features from queries and passages. There are pre-build proprietary models like [Open AI embedding API](https://platform.openai.com/docs/guides/embeddings) and [open-sourced models](https://huggingface.co/spaces/mteb/leaderboard) to help you encode queries and passages into embeddings. Using embedding also allows you to search multi-modality data.

One advantage of the DR retrieval models in comparison with SR algorithms is the ability to train them for specific tasks. You can fine-tune embedding models to better align with your applications to improve DR. It’s essential to define “similarity” in the context of your applications. For example, if “a pair of Nike shoes” differs significantly from “a pair of Adidas tennis shoes,” you can generate training data to reflect that relation. The embedding models normally employ [triplet Loss and siamese neural networks](https://doordash.engineering/2021/09/08/using-twin-neural-networks-to-train-catalog-item-embeddings/) to maximize the cosine (or dot product) distances between negative input pairs and minimize the distance between positive pairs. The balance between positive, simple negative, and hard negative pairs is an interesting topic in training / fine-tuning embedding models. [Sentence-transformers](https://huggingface.co/blog/how-to-train-sentence-transformers) is a great framework to use to train/fine-tune embedding models. In addition, we see more instruction embedding like [FlagEmbedding](https://huggingface.co/BAAI/bge-large-en) being invented, which gives us more flexibility to use the same model for different downstream tasks. For example, we can add the prefix “query: ” to indicate we want to encode the query in the prompt.

Using embedding is not a silver bullet in retrieval, and sometimes, [it can not match the classic BM25 baseline’s performance without further tuning](https://arxiv.org/pdf/2212.03533.pdf). If you’re looking for the best retrieval performance, hybrid approaches combining SR and DR are state-of-the-art. You can use [Reciprocal Rank Fusion (RRF)](https://www.elastic.co/guide/en/elasticsearch/reference/current/rrf.html) to combine results across approaches with different relevance indicators into a single result set.

The hybrid approaches and RRF may lead to latency overhead, because there are multi compute heavy workloads (for example ML models) involved during the process. Luckily, there are some methods to alleviate the performance issue. [Amazon presents a four-step process to adapt BERT-like models in search](https://www.amazon.science/publications/web-scale-semantic-product-search-with-large-language-models). The team trained a distilled embedding model, which only introduced extra 4 ms latency while improving retrieval and business metrics.

Other improvement method discussions, like metadata filtering, can be found in Appendix.

### 3. Have more recent information available in context

Great. Now that we have covered # 1 and # 2, we can move to the last requirement, the interactive query. When Tom is browsing a product’s web page, the shopping assistant should be able to know Tom’s current status to give appropriate feedback. The click stream information could be stored in a NoSQL DB like MongoDB or Redis. To implement the requirement, we need to have a low-latency data pipeline to ingest click stream data to our store, and our retriever should also be able to fetch data from the store in a low-latency fashion. The ingestion part is more of a data engineering problem, and the discussion is out of the scope of this blog. For the data read, you can use the [Program-aided language model (PAL) chain](https://python.langchain.com/docs/use_cases/code_writing/pal) or [Function calling](https://openai.com/blog/function-calling-and-other-api-updates) to ask LLMs to decide if it’s required to fetch the latest information from storage. For example, LLM can convert “Who are my top ten customers today?” to an internal API call, such as

```
get_customers_by_revenue(start_datetime: string, end_datetime: string, limit: int)
```

, or “How many orders did Acme, Inc. place last minute?” to a SQL query using

```
sql_query(query: string)
```

### 4. Augmenting retrieval results to support prompt engineering

After retrieving related passages from different data stores, we need to wrap them up into prompts to send them to the following generator. It is also essential to include the retrieval channel in the prompt to ensure the LLMs understand the source of information. For one of Tom’s queries, we may end up with passages from product information, customer review, and his recent purchase. The generator should be able to tell which information is more relevant to the current query. Timestamp-related information is also useful for LLMs to filter out stale information.

## Conclusion

In summary, the idea of crafting an efficient RAG application goes beyond the confines of an LLM's abilities. It becomes a nuanced process of fetching, interpreting, and effectively utilizing a distinct collection of information with the assistance of a skilled retriever. A precise fusion of information retrieval techniques and response generation makes RAG an innovative and effective tool for handling tasks requiring specific knowledge.

As we wind up, it’s clear there is an expanding list of complex techniques continually being developed to boost retrieval performance, such as the potential use of [graph DB](https://gpt-index.readthedocs.io/en/latest/examples/index_structs/knowledge_graph/KnowledgeGraphDemo.html), or [leveraging user behavior data to complement query-item semantic matching](https://recsys.substack.com/p/llama-e-for-e-commerce-authoring#%C2%A7beyond-semantics-learning-a-behavior-augmented-relevance-model-with-self-supervised-learning). As the field rapidly evolves, establishing well-defined success measurements and creating a solid performance baseline cannot be overstated, especially when exploring new use cases. May this blog invoke thoughts when you build your next RAG applications!

Thank you, [GPT-4](https://openai.com/gpt-4)and [Caude 2](https://www.anthropic.com/index/claude-2), for the advice to make the content more engaging and concise. I learned a lot from collaborating with you when writing the blog. TLDR of this blog was also generated by GPT-4.

## Reference:

- How to get optimal retrieval performance with vector search: https://www.elastic.co/blog/lexical-ai-powered-search-elastic-vector-database

- Ask like a human: Implementing semantic search on Stack Overflow: https://stackoverflow.blog/2023/07/31/ask-like-a-human-implementing-semantic-search-on-stack-overflow/#:~:text=Weaviate%2C%20a%20startup%20focused%20on%20building%20open%20source%20AI%2Dfirst%20infrastructure%2C%20satisfied%20all%20those%20requirements.%20So%20far%2C%20so%20good

- Knowledge Retrieval Architecture for LLM’s (2023): https://mattboegner.com/knowledge-retrieval-architecture-for-llms/

## Appendix:

There are some considerations to using SR in RAG.

- Choosing the appropriate document size. If the document size is too big, including all information in the prompt is hard. We don’t have enough context if the document size is too small. One method that can help is using LLM to summarise a document and index the summarised documents and raw documents, which would introduce extra latency during indexing and add risks to introducing hallucination.

- Increasing the probability of matching query to passages. Query Rewriting is a method using LLM to increase the possibility of the lexical match between query and passages. For example, the original query is “best Android phone under 300 dollars”. We have the following new queries after asking an LLM to rewrite the question. “What is the best Android smartphone I can buy for under $300?” and “Please recommend the top Android mobile phone priced under 300 dollars with strong battery life and camera.” After that, we can use the augmented queries with SR methods to perform the search, which gives us a better chance to identify candidate records. Fuzzy queries are another well-used feature.

- Handling synonyms in your domain language. Search engines provide various functions to include the keyword match. For example, Elastic Search has synonym filters to help you define customer dictionaries.

- Hypothetical Document Embeddings (HyDE) approach can also be used to improve retrieval performance. In that case, a user would not search for a vector close to the question being posed but instead for a vector close to a hypothetical answer to the query. The hypothetical answer could, in this case, be generated by a LLM. The core assumption is a generated answer is often closer to the embedding one searches for rather than the question one asks.

In addition to matching text, keywords, and embeddings, document metadata is also crucial. It can match information and improve retrieval performance by using it as a filter. The metadata can include information like source file name, category of documents, keywords, and the position of the current document in the corpus. LlamaIndex provides [helper functions](https://gpt-index.readthedocs.io/en/latest/core_modules/data_modules/documents_and_nodes/usage_documents.html) to extract, store, and query these metadata. Another advanced trick is using LLM (or [purposed-built NLP models](https://www.ayadata.ai/blog-posts/what-is-named-entity-recognition-in-nlp/#:~:text=NER%20is%20a%20critical%20subfield,or%20a%20date%20or%20time.)) to extract metadata during indexing and then leverage them during the query. For example, you can remove (or infer) the product scene (.i.e, “party”, “BBQ in summer”) from the product description (.i.e, “coal for a kebab” ). [langchain’s Tagging](https://python.langchain.com/docs/use_cases/tagging) feature could be useful if you want to use LLMs to extract metadata. But this may slow down the indexing process and may incur significant costs. Training a custom purpose-built light NLP model could be more efficient if LLM is too heavy.
