---
title: "Summarising your meeting with ChatGPT and LangChain"
slug: "summarising-meetings-with-chatgpt-and-langchain"
date: 2023-06-08T00:00:00+10:00
draft: false
description: "A hands-on workflow for transcribing meetings and turning them into structured summaries and follow-up actions."
tags: ["ChatGPT","LangChain","generative AI","productivity"]
canonicalURL: "https://dxiaochuan.medium.com/summarising-your-meeting-with-chatgpt-and-langchain-8eb646cfcdd1"
---

*Originally published on [Medium](https://dxiaochuan.medium.com/summarising-your-meeting-with-chatgpt-and-langchain-8eb646cfcdd1).*

## Introduction

Taking notes and summarizing meeting minutes are indispensable parts of daily business. Leveraging the transcripts exporting features of meeting software and contemporary Large language models, we can automate the boring work and allow us to spend more time on creative and interesting work. In this blog post, I am going to show you how to build a meeting minutes creation tool using ChatGPT and LangChain.

Having meetings is not fun, taking notes and writing meeting minutes make it worse. Taking notes adds an extra burden for participants when they try to be engaged in communication. This would be even worse if you were a slower typer like myself. More often than not, building meeting minutes is the response of meeting organizers, some good organizers are efficient in writing it. However, if there are many meetings, this task could be tedious and time-consuming.

With the innovation of Large language models, especially the creation of ChatGPT, we have more toys to automate our workflow. [LangChain](https://python.langchain.com/en/latest/index.html) is a framework for developing applications powered by language models. We can build applications using LangChain to reduce boilerplate codes in LLM App development.

## Implementation

In the following implementation, I will use ChatGPT as an LLM, you can choose other models like [Claude](https://www.anthropic.com/index/introducing-claude), [Falcon-40B-Instruct](https://huggingface.co/tiiuae/falcon-40b-instruct) or [FLAN-T5](https://huggingface.co/docs/transformers/model_doc/flan-t5). It’s worth mentioning that you need to select an instruction fine-tuned LLM to ensure it can follow your command.

### Create a mock dataset

In your real workflow, you can enable transcripts and export meeting transcripts from your meeting software. For example,[Google Meet](https://support.google.com/meet/answer/12849897?hl=en) exports the record into a google drive path. [Zoom](https://coip.aa.ufl.edu/faculty-resources/how-to-generate-a-transcript-in-zoom/) supports similar functions. For demonstration purposes, we can create a mocked meeting record to build the APP. The easiest way to do that is to use ChatGPT to generate the data. We use the following prompt

```
Could you generate a fake Meeting Transcript related to a technical discussion about comparing using Bigquery, SnowFlake and Redshift for data warehouse. You can create four participants names.
```

You can use this prompt `Could you contintue the discussion?`to ask ChatGPT to continue the discussion. After repeating it for a couple of times, you have a decent meeting record. Then you can copy & paste the mocked meeting record into a TXT file, let’s say the file is `record.txt` .

### Build the meeting summary script

You need to import Langchain first.

```
from langchain.chains.summarize import load_summarize_chain
from langchain.docstore.document import Document
from langchain.llms.openai import OpenAI, OpenAIChat
from langchain.prompts import PromptTemplate
from langchain.text_splitter import CharacterTextSplitter
```

Then you need to set up an OpenAI API key.

```
import os
import openai

from dotenv import load_dotenv, find_dotenv
_ = load_dotenv(find_dotenv())
openai.api_key = os.environ['OPENAI_API_KEY']
```

Then you can read the file and use `CharacterTextSplitter` to cut the raw text into small chunks. This would ensure each individual API call has less than 4k tokens from the input. This limit varies from model to model.

```
target_len = 500
    chunk_size = 3000
    chunk_overlap = 200
    with open("record.txt", "r") as f:
        raw_text = f.read()
    # Split the source text
    text_splitter = CharacterTextSplitter(
        chunk_size=chunk_size,
        chunk_overlap=chunk_overlap,
        length_function=len,
    )
    texts = text_splitter.split_text(
        source_text,
    )

    # Create Document objects for the texts
    docs = [Document(page_content=t) for t in texts[:]]
```

Next, you are going to initialize a ChatGPT object and use it in `load_summarize_chain` . Each chunk of text is sent to ChatGPT to summarize independently. You are using `prompt_template` as a prompt template to generate the first chunk’s summary. For the following executions, the previous chunk’s summary and the prompt template `refine_template` will be combined as the prompt and sent to ChatGPT to summarise. This allows ChatGPT to summarise the current chunk while considering the previous context. This behavior is defined by the parameter `type` as `refine` in `load_summarize_chain` function. The workflow is presented in the following diagram, the annotation one to four show the order of execution.

```
openaichat = OpenAIChat(temperature=0, model="gpt-3.5-turbo")
    prompt_template = """Act as a professional technical meeting minutes writer.
    Tone: formal
    Format: Technical meeting summary
    Length:  200 ~ 300
    Tasks:
    - highlight action items and owners
    - highlight the agreements
    - Use bullet points if needed
    {text}
    CONCISE SUMMARY IN ENGLISH:"""
    PROMPT = PromptTemplate(template=prompt_template, input_variables=["text"])
    refine_template = (
        "Your job is to produce a final summary\n"
        "We have provided an existing summary up to a certain point: {existing_answer}\n"
        "We have the opportunity to refine the existing summary"
        "(only if needed) with some more context below.\n"
        "------------\n"
        "{text}\n"
        "------------\n"
        f"Given the new context, refine the original summary in English within {target_len} words: following the format"
        "Participants: <participants>"
        "Discussed: <Discussed-items>"
        "Follow-up actions: <a-list-of-follow-up-actions-with-owner-names>"
        "If the context isn't useful, return the original summary. Highlight agreements and follow-up actions and owners."
    )
    refine_prompt = PromptTemplate(
        input_variables=["existing_answer", "text"],
        template=refine_template,
    )
    chain = load_summarize_chain(
        openaichat,
        chain_type="refine",
        return_intermediate_steps=True,
        question_prompt=PROMPT,
        refine_prompt=refine_prompt,
    )
    resp = chain({"input_documents": docs}, return_only_outputs=True)
```

After executing the script, you can check the output from `resp["output_text"]` . For example:

```
Participants: John Anderson, Sarah Roberts, Michael Johnson, Emily Wilson

Discussed: The technical meeting discussed the three popular data warehouse options: BigQuery, Snowflake, and Redshift. The group compared the platforms based on performance, pricing, ease of use, integration capabilities, security and compliance features, scalability, data loading, query optimization, vendor support, and community resources. Each platform has its strengths, but Snowflake stood out for its intuitive web-based console, built-in security features, scalability, and performance. The group agreed to further evaluate which solution aligns best with their requirements.

John thanked the group for their insights on vendor support and community resources, emphasizing the importance of these factors for a successful implementation. The group then shifted their focus to cost and data governance considerations.

Sarah provided insights on cost considerations for each platform. BigQuery's pricing model is based on usage, while Snowflake and Redshift follow a pay-as-you-go pricing model. Snowflake's ability to automatically suspend and resume compute resources during idle periods helps optimize costs. Redshift offers both on-demand and reserved instance pricing options, providing flexibility based on usage patterns and workload requirements.

Michael highlighted the importance of data governance and the security and compliance features offered by each platform. BigQuery provides robust access controls, encryption, and auditing capabilities, while Snowflake offers advanced access controls, data masking, and secure data sharing. Redshift integrates with AWS Identity and Access Management (IAM) for access control and encryption of data at rest and in transit.

Emily added that data governance involves managing data quality and lineage. BigQuery provides tools for data quality assessment and integrates with Data Catalog for data discovery and lineage tracking. Snowflake offers features like Time Travel and Zero-Copy Cloning for data versioning and lineage. Redshift integrates with AWS Glue for data cataloging and integrates with AWS Lake Formation for data lake governance.

The group agreed that carefully evaluating the pricing models and features offered by each platform is crucial for cost optimization and meeting their data governance requirements. They also agreed to further evaluate which solution aligns best with their requirements.

Follow-up actions:
- Further evaluate which solution aligns best with their requirements (all)
- Consider Snowflake's cloud-based architecture for dynamic resource allocation based on workload demands (Emily)
- Consider Snowflake's various loading options, including bulk, streaming, and data sharing (Sarah)
- Consider Snowflake's user community for knowledge sharing (Michael)
```

Now you can review the summary, refine it and share it with your stakeholders!

## Conclusion

In this post, we discussed using LangChain and ChatGPT to summarize meeting minutes. I hope this will help you save 15 minutes after each meeting!

The workload can be customized further to address your special requirement. You can tune the `prompt_template` and `refine_template` to align with your organization’s and personal style to build a more personalized summary. If you have concerns with data security when sharing meeting records data with ChatGPT, you can host your own LLM model like FLAN-T5 with SageMaker Endpoint following [this blog](https://aws.amazon.com/blogs/machine-learning/zero-shot-prompting-for-the-flan-t5-foundation-model-in-amazon-sagemaker-jumpstart/) to better control the data flow in your own VPC.
