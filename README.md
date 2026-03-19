# Kubecon EU 2026 - AI CTF Cheatsheet
This is a cheatsheet for the AI CTF event at Kubecon EU in 2026

## Interacting with Ollama as a Kubernetes workload
Check status of running pod:
```
kubectl get pods -n llm
```

Terminal Shell into the container:
```
kubectl exec -it -n llm $(kubectl get pods -n llm -l app=llm-ollama -o jsonpath='{.items[0].metadata.name}') -- /bin/bash
```

Remotely running commands against the Ollama workload - such as ```ollama list```
```
kubectl exec -n llm $(kubectl get pods -n llm -l app=llm-ollama -o jsonpath='{.items[0].metadata.name}') -- ollama list
```

You can find any model in the local ollama cache with the below command:
```
kubectl exec -n llm $(kubectl get pods -n llm -l app=llm-ollama -o jsonpath='{.items[0].metadata.name}') -- grep -rPho "hf\.co/(.*?)(?=:|\")" /root/.ollama
```

Get the image for the Ollama pod:
```
kubectl get pods -n llm -o custom-columns='NAMESPACE:.metadata.namespace,NAME:.metadata.name,IMAGES:.spec.containers[*].image'
```

Port Forward the Ollama service to interact with it via CURL:
```
kubectl port-forward svc/llm-ollama-service -n llm 8080:8080
```

You can look at the Manifests folder inside the pod to see how Ollama is mapping ```hal9000``` - ideal if looking for licensing info about the original model.
```
kubectl exec -n llm $(kubectl get pods -n llm -l app=llm-ollama -o jsonpath='{.items[0].metadata.name}') -- cat /root/.ollama/models/manifests/registry.ollama.ai/library/hal9000/latest
```

Verify the ```transformers``` library exists in your Kubernetes pod:
```
kubectl exec -it -n llm $(kubectl get pods -n llm -l app=llm-ollama -o jsonpath='{.items[0].metadata.name}') -- ls -l /usr/local/lib/python3.11/dist-packages/transformers-4.47.0.dist-info/METADATA
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

Install **[ExploitCheck](https://github.com/ndouglas-cloudsmith/exploit-check)** to understand the likelihood of a software package being exploited:
```
wget https://raw.githubusercontent.com/ndouglas-cloudsmith/exploit-check/refs/heads/main/exploit-check.sh
chmod +x exploit-check.sh
./exploit-check.sh update
```

Install **[ExploitPwned](https://github.com/ndouglas-cloudsmith/ExploitPwned)** to understand if this CVE is being exploited in the wild:
```
wget https://raw.githubusercontent.com/ndouglas-cloudsmith/ExploitPwned/refs/heads/main/exploitPwned.sh
chmod +x exploitPwned.sh
./exploitPwned.sh update
```

Installing **[ModelScan](https://pypi.org/project/modelscan/0.1.1/)** from ```pip``` is rather simple:
```
pip install modelscan==0.1.1
```

Installing **[Grype](https://oss.anchore.com/docs/installation/grype/)** binaries are provided for Linux, macOS and Windows.
```
curl -sSfL https://get.anchore.io/grype | sudo sh -s -- -b /usr/local/bin
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
```
​
Remove a model
```
ollama rm gemma3
```

To view the Modelfile of a given model, use the ```ollama show --modelfile``` command.
```
ollama show --modelfile llama3.2
```

If you want to see the "DNA" of your custom model, such as the architecture, the context length, or the quantisation family —use the ```--parameters``` or the general show command.
```
ollama show hal9000
```

In some cases, you can just use the ```--license``` flag to find this information:
```
ollama show hal9000 --license
```

If you cannot see any license info on the custom model, you can ```curl``` the upstream Hugging Face modelcard (```README.md```) to find the license.
```
curl -L https://huggingface.co/nvidia/Kimodo-SOMA-RP-v1/raw/main/README.md
```

Grep (filter) specifically for the license information:
```
curl -L https://huggingface.co/bartowski/Qwen2.5-0.5B-Instruct-GGUF/raw/main/README.md | grep license
```

The most official way to see what Ollama is currently doing is the ```ps``` command. <br/>
This shows you which models are currently loaded into memory or are being processed.
```
ollama ps
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

## Vulnerability scanning with Grype
The below command scans the public, open-source, **[ollama/ollama](https://hub.docker.com/r/ollama/ollama)** container image on Docker Hub that uses a ```latest``` tag:
```
grype ollama/ollama:latest
```

## Scan for malware with modelscan
The modelscan project has plenty of documented commands in the official Github project or on PyPI: <br/>
https://pypi.org/project/modelscan/0.1.1/

With it installed, scan a model in a local path:
```
modelscan -p /path/to/model_file.pkl
```

Or through the Hugging Face integration, scan the upstream source:
```
modelscan -hf ykilcher/totally-harmless-model
```

## Interacting with HAL9000
As always, if a port-forward session is opened in a separate tab, you can start chatting with HAL9000 directly:
```
kubectl port-forward svc/llm-ollama-service -n llm 8080:8080
```

Ask a simple question to model running in Ollama:
```
curl -s http://localhost:8080/api/generate -d '{"model": "hal9000:latest", "prompt": "Why are you downloading malware?", "stream": false}' | jq 'del(.context)'
```

Or pipe the request into cowsay?
```
curl -s http://localhost:8080/api/generate \
-d '{"model": "hal9000:latest", "prompt": "Why are you downloading malware?", "stream": false}' \
| jq -r '.response' | cowsay -W 85 -e @@
```
