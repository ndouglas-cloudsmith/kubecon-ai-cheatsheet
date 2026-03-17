# Kubecon EU 2026 - AI CTF Cheatsheet
This is a cheatsheet for the AI CTF event at Kubecon EU in 2026

## Interacting with Ollama as a Kubernetes workload
Terminal Shell into the container:
```
kubectl exec -it -n llm $(kubectl get pods -n llm -l app=llm-ollama -o jsonpath='{.items[0].metadata.name}') -- /bin/bash
```

## Install Tools

Install **[Ollama](https://ollama.com/)** to run your open-source LLM models:
```
curl -fsSL https://ollama.com/install.sh | sh
```

Install the **[Hugging Face CLI](https://huggingface.co/docs/huggingface_hub/en/guides/cli)** to download your LLM models and datasets.
```
curl -LsSf https://hf.co/cli/install.sh | bash -s -- --force
source ~/.bashrc
hf --help
```

Install **[Trufflehog](https://github.com/trufflesecurity/trufflehog)** to find sensitive credentials exposed in Hugging Face Hub.
```
curl -sSfL https://raw.githubusercontent.com/trufflesecurity/trufflehog/main/scripts/install.sh | sh -s -- -b /usr/local/bin
```

## Ollama CLI commands
We will use Ollama as the runtime for our LLM models in this CTF, so its good to familiarise ourselves with the CLI commands: <br/>
https://docs.ollama.com/cli
<br/><br/>
```
ollama run embeddinggemma "Hello world"
```
Output is a JSON array:
```
echo "Hello world" | ollama run nomic-embed-text
```
​Download a model
```
ollama pull gemma3
```​
Remove a model
```
ollama rm gemma3
```
To view the Modelfile of a given model, use the ```ollama show --modelfile``` command.
```
ollama show --modelfile llama3.2
```

## Hugging Face CLI commands
The best way to get familiar with the Hugging Face CLI to check the official docs: <br/>
https://huggingface.co/docs/huggingface_hub/guides/search#using-the-cli
<br/><br/>
List ```models```:
```
hf models ls --author=HuggingFaceTB --limit=10
```
Get info about a ```specific model``` on the hub:
```
hf models info Qwen/Qwen-Image-2512
```
List ```datasets```:
```
hf datasets ls --filter "format:parquet" --sort=downloads
```
Get info about a ```specific dataset``` on the hub:
```
hf datasets info HuggingFaceFW/fineweb
```
List ```Spaces```:
```
hf spaces ls --search "3d"
```
Get info about a specific ```Space``` on the hub:
```
hf spaces info enzostvs/deepsite
```
We'll come to this later, but [Skills](https://github.com/huggingface/skills) are quickly becoming the biggest attack surface when we start talking about AI agents.
```
hf skills
```

To download a **single file** from a repo, simply provide the ```repo_id``` and ```filename``` as follows:
```
hf download gpt2 config.json
```

The command will always print on the last line **the path to the file** on your local machine. <br/>
You can read the entire contents of the local cache with the below command:
```
ls -R /root/.cache/huggingface/hub/
```

To download a file located in a subdirectory of the repo, you should provide the path of the file in the repo in posix format like this:
```
hf download HiDream-ai/HiDream-I1-Full text_encoder/model.safetensors
```

You can check this using the ```--dry-run``` parameter. It lists all files to download on the repo and checks whether they are already downloaded or not. <br/>
This gives an idea of how many files have to be downloaded and their sizes.
```
hf download openai-community/gpt2 --dry-run
```


## Sensitive credential exposure with Trufflehog
The Truffle Security team have a good blog post on how to use the tool once installed: <br/>
https://trufflesecurity.com/blog/trufflehog-partners-with-hugging-face-to-scan-for-secrets

Scan a Hugging Face Model
```
trufflehog huggingface --model <model_id>
```
Scan a Hugging Face Dataset
```
trufflehog huggingface --dataset <dataset_id>
```
Scan a Hugging Face Space
```
trufflehog huggingface --space <space_id>
```
