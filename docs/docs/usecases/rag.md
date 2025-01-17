# Basic RAG

Here we show an example of building a basic RAG application with Indexify. We are going to upload content about Kevin Durant from Wikipedia and ask questions about KD's career.

### Install the Indexify Extractor SDK and the Indexify Client
```bash
pip install indexify-extractor-sdk indexify
```

### Start the Indexify Server
```bash
indexify server -d
```

### Download an Embedding Extractor
On another terminal start the embedding extractor which we will use to index text from the wikiepdia page.
```bash
indexify-extractor download hub://embedding/minilm-l6
indexify-extractor join minilm_l6:MiniLML6Extractor
```

### Upload Content
We will use the lanchain wikipedia loader to download content from wikipedia and upload to Indexify. We will also use langchain to prompt OpenAI for the RAG application.
```python
pip install --upgrade --quiet  wikipedia langchain_openai
```

Now download some pages from Wikipedia and upload them to Indexify
```python
from langchain_community.document_loaders import WikipediaLoader
docs = WikipediaLoader(query="Kevin Durant", load_max_docs=10).load()
```

Instantiate the Indexify Client 
```python
from indexify import IndexifyClient
client = IndexifyClient()
```

```python
client = IndexifyClient()
for doc in docs:
    client.add_documents(doc.page_content)
```

### Create an Extraction Policy 
```python
client.add_extraction_policy(extractor='tensorlake/minilm-l6', name="wiki-embedding")
```

### Perform RAG

Initialize the Langchain Retreiver, create a chain to prompt OpenAI with data retreived from Indexify to create a simple Q and A bot
```python
from langchain_core.output_parsers import StrOutputParser
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough
from langchain_openai import ChatOpenAI
```

```python
template = """Answer the question based only on the following context:
{context}

Question: {question}
"""
prompt = ChatPromptTemplate.from_template(template)

model = ChatOpenAI()

chain = (
    {"context": retriever, "question": RunnablePassthrough()}
    | prompt
    | model
    | StrOutputParser()
)
```
Now ask any question about KD -
```python
chain.invoke("When and where did KD win his championships?")
```

```bash
'Kevin Durant won his championships with the Golden State Warriors in 2017 and 2018.'
```


