# rag-intro

## Preparing the Environment

In codespaces:

Create a repository, e.g. "llm-zoomcamp-rag-workshop"
Start a codespace there

Let's install it:
```bash
pip install pipenv
```

Install the packages:
```bash
pipenv install tqdm notebook==7.1.2 openai elasticsearch
```

Create a new OpenAI key, copy it.

Option 1: put the key to an env variable:
```bash
export OPENAI_API_KEY="TOKEN"
```

Option 2: But a better way for managing keys is using direnv:
```bash
sudo apt update
sudo apt install direnv 
direnv hook bash >> ~/.bashrc
```

Create / edit .envrc in your project directory and add to it:
```
export OPENAI_API_KEY='sk-proj-key'
```

Make sure .envrc is in your .gitignore - never commit it!
```bash
echo ".envrc" >> .gitignore
```

Allow direnv to run:
```bash
direnv allow
```

Start a new terminal, and there run jupyter:
```bash
pipenv run jupyter notebook
```

In another terminal, run elasticsearch with docker:
```bash
docker run -it \
    --rm \
    --name elasticsearch \
    -p 9200:9200 \
    -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    docker.elastic.co/elasticsearch/elasticsearch:8.4.3
```

If you get "Elasticsearch has quit unexpectedly", give it more RAM:
```bash
docker run -it \
    --rm \
    --name elasticsearch \
    -m 2G \
    -p 9200:9200 \
    -p 9300:9300 \
    -e "discovery.type=single-node" \
    -e "xpack.security.enabled=false" \
    docker.elastic.co/elasticsearch/elasticsearch:8.4.3
```

Verify that ES is running
```bash
curl http://localhost:9200
```

You should get something like this:
```bash
{
  "name" : "63d0133fc451",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "AKW1gxdRTuSH8eLuxbqH6A",
  "version" : {
    "number" : "8.4.3",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "42f05b9372a9a4a470db3b52817899b99a76ee73",
    "build_date" : "2022-10-04T07:17:24.662462378Z",
    "build_snapshot" : false,
    "lucene_version" : "9.3.0",
    "minimum_wire_compatibility_version" : "7.17.0",
    "minimum_index_compatibility_version" : "7.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## Retrieval

RAG consists of multiple components, and the first is R - "retrieval". 

For retrieval, we need a search system. In our example, we will use elasticsearch for searching.

## Searching in the documents
Create a nootebook "elastic_rag" or something like that. We will use it for our experiments. Follow elastic_rag.ipynb of this repo. 

## Generation - Answering questions

Now let's do the "G" part - generation based on the "R" output
Keep following elastic_rag.ipynb of this repo. 





Building a Prompt
Now let's build a prompt. First, we put all the documents together in one string:

context_template = """
Section: {section}
Question: {question}
Answer: {text}
""".strip()

context_docs = retrieve_documents(user_question)

context_result = ""

for doc in context_docs:
    doc_str = context_template.format(**doc)
    context_result += ("\n\n" + doc_str)

context = context_result.strip()
print(context)
Now build the actual prompt:

prompt = f"""
You're a course teaching assistant. Answer the user QUESTION based on CONTEXT - the documents retrieved from our FAQ database. 
Only use the facts from the CONTEXT. If the CONTEXT doesn't contan the answer, return "NONE"

QUESTION: {user_question}

CONTEXT:

{context}
""".strip()
Now we can put it to OpenAI API:

response = client.chat.completions.create(
    model="gpt-4o",
    messages=[{"role": "user", "content": prompt}]
)
answer = response.choices[0].message.content
answer
Note: there are system and user prompts, we can also experiment with them to make the design of the prompt cleaner.

Cleaning
Now let's put everything together in one function:

context_template = """
Section: {section}
Question: {question}
Answer: {text}
""".strip()

prompt_template = """
You're a course teaching assistant.
Answer the user QUESTION based on CONTEXT - the documents retrieved from our FAQ database.
Don't use other information outside of the provided CONTEXT.  

QUESTION: {user_question}

CONTEXT:

{context}
""".strip()


def build_context(documents):
    context_result = ""
    
    for doc in documents:
        doc_str = context_template.format(**doc)
        context_result += ("\n\n" + doc_str)
    
    return context_result.strip()


def build_prompt(user_question, documents):
    context = build_context(documents)
    prompt = prompt_template.format(
        user_question=user_question,
        context=context
    )
    return prompt

def ask_openai(prompt, model="gpt-4o"):
    response = client.chat.completions.create(
        model=model,
        messages=[{"role": "user", "content": prompt}]
    )
    answer = response.choices[0].message.content
    return answer

def qa_bot(user_question):
    context_docs = retrieve_documents(user_question)
    prompt = build_prompt(user_question, context_docs)
    answer = ask_openai(prompt)
    return answer
Now we can ask it different questions

qa_bot("I'm getting invalid reference format: repository name must be lowercase")
qa_bot("I can't connect to postgres port 5432, my password doesn't work")
qa_bot("how can I run kafka?")
