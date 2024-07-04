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

## What's next
Use Open-Souce
Build an interface, e.g. streamlit
Deploy it

cf. [this repo](https://github.com/alexeygrigorev/llm-rag-workshop/blob/main/README.md)
