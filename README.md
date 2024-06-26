# Mistral Inference
<a target="_blank" href="https://colab.research.google.com/github/mistralai/mistral-inference/blob/main/tutorials/getting_started.ipynb">
  <img src="https://colab.research.google.com/assets/colab-badge.svg" alt="Open In Colab"/>
</a>


This repository contains minimal code to run our 7B, 8x7B and 8x22B models.

Blog 7B: [https://mistral.ai/news/announcing-mistral-7b/](https://mistral.ai/news/announcing-mistral-7b/)\
Blog 8x7B: [https://mistral.ai/news/mixtral-of-experts/](https://mistral.ai/news/mixtral-of-experts/)\
Blog 8x22B: [https://mistral.ai/news/mixtral-8x22b/](https://mistral.ai/news/mixtral-8x22b/)

Discord: [https://discord.com/invite/mistralai](https://discord.com/invite/mistralai)\
Documentation: [https://docs.mistral.ai/](https://docs.mistral.ai/)\
Guardrailing: [https://docs.mistral.ai/usage/guardrailing](https://docs.mistral.ai/usage/guardrailing)

## Installation

### PyPI

```
pip install mistral-inference
```

### Local

```
cd $HOME && git clone https://github.com/mistralai/mistral-inference
cd $HOME/mistral-inference && poetry install .
```

## Model download

| Name        | Download | md5sum |
|-------------|-------|-------|
| 7B Instruct v3 | https://models.mistralcdn.com/mistral-7b-v0-3/mistral-7B-Instruct-v0.3.tar | `80b71fcb6416085bcb4efad86dfb4d52` |
| 8x7B Instruct | https://models.mistralcdn.com/mixtral-8x7b-v0-1/Mixtral-8x7B-v0.1-Instruct.tar (**Updated model coming soon!**) | `8e2d3930145dc43d3084396f49d38a3f` |
| 8x22 Instruct | https://models.mistralcdn.com/mixtral-8x22b-v0-3/mixtral-8x22B-Instruct-v0.3.tar | `471a02a6902706a2f1e44a693813855b` |
| 7B Base | https://models.mistralcdn.com/mistral-7b-v0-3/mistral-7B-v0.3.tar | `0663b293810d7571dad25dae2f2a5806` |
| 8x7B |     **Updated model coming soon!**       | - |
| 8x22B | https://models.mistralcdn.com/mixtral-8x22b-v0-3/mixtral-8x22B-v0.3.tar | `a2fa75117174f87d1197e3a4eb50371a` |

Note: 
- **Important**:
  - `mixtral-8x22B-Instruct-v0.3.tar` is exactly the same as [Mixtral-8x22B-Instruct-v0.1](https://huggingface.co/mistralai/Mixtral-8x22B-Instruct-v0.1), only stored in `.safetensors` format
  - `mixtral-8x22B-v0.3.tar` is the same as [Mixtral-8x22B-v0.1](https://huggingface.co/mistralai/Mixtral-8x22B-v0.1), but has an extended vocabulary of 32768 tokens.
- All of the listed models above supports function calling. For example, Mistral 7B Base/Instruct v3 is a minor update to Mistral 7B Base/Instruct v2,  with the addition of function calling capabilities. 
- The "coming soon" models will include function calling as well. 
- You can download the previous versions of our models from our [docs](https://docs.mistral.ai/getting-started/open_weight_models/#downloading).


Create a local folder to store models
```sh
export MISTRAL_MODEL=$HOME/mistral_models
mkdir -p $MISTRAL_MODEL
```

Download any of the above links and extract the content, *e.g.*:

```sh
export M7B_DIR=$MISTRAL_MODEL/7B_instruct
wget https://models.mistralcdn.com/mistral-7b-v0-3/mistral-7B-Instruct-v0.3.tar
mkdir -p $M7B_DIR
tar -xf mistral-7B-Instruct-v0.3.tar -C $M7B_DIR
```

or 

```sh
export M8x7B_DIR=$MISTRAL_MODEL/8x7b_instruct
wget https://models.mistralcdn.com/mixtral-8x7b-v0-1/Mixtral-8x7B-v0.1-Instruct.tar
mkdir -p $M8x7B_DIR
tar -xf Mixtral-8x7B-v0.1-Instruct.tar -C $M8x7B_DIR
```

## Usage

The following sections give an overview of how to run the model from the Command-line interface or from Python.

### CLI

- **Demo**

To test that a model works in your setup, you can run the `mistral-demo` command.
The 7B models can be tested on a single GPU as follows:

```sh
mistral-demo $M7B_DIR
```

Large models, such **8x7B** and **8x22B** have to be run in a multi-GPU setup.
For these models, you can use the following command:

```sh
torchrun --nproc-per-node 2 --no-python mistral-demo $M8x7B_DIR
```

*Note*: Change `--nproc-per-node` to more GPUs if available.

- **Chat**

To interactively chat with the models, you can make use of the `mistral-chat` command.

```sh
mistral-chat $M7B_DIR --instruct
```

For large models, you can make use of `torchrun`.

```sh
torchrun --nproc-per-node 2 --no-python mistral-chat $M8x7B_DIR --instruct
```

*Note*: Change `--nproc-per-node` to more GPUs if necessary (*e.g.* for 8x22B).

### Python

- *Instruction Following*:

```py
from mistral_inference.model import Transformer
from mistral_inference.generate import generate

from mistral_common.tokens.tokenizers.mistral import MistralTokenizer
from mistral_common.protocol.instruct.messages import UserMessage
from mistral_common.protocol.instruct.request import ChatCompletionRequest


tokenizer = MistralTokenizer.from_file("./mistral_7b_instruct/tokenizer.model.v3")  # change to extracted tokenizer file
model = Transformer.from_folder("./mistral_7b_instruct")  # change to extracted model dir

completion_request = ChatCompletionRequest(messages=[UserMessage(content="Explain Machine Learning to me in a nutshell.")])

tokens = tokenizer.encode_chat_completion(completion_request).tokens

out_tokens, _ = generate([tokens], model, max_tokens=64, temperature=0.0, eos_id=tokenizer.instruct_tokenizer.tokenizer.eos_id)
result = tokenizer.instruct_tokenizer.tokenizer.decode(out_tokens[0])

print(result)
```

- *Function Calling*:

```py
from mistral_common.protocol.instruct.tool_calls import Function, Tool

completion_request = ChatCompletionRequest(
    tools=[
        Tool(
            function=Function(
                name="get_current_weather",
                description="Get the current weather",
                parameters={
                    "type": "object",
                    "properties": {
                        "location": {
                            "type": "string",
                            "description": "The city and state, e.g. San Francisco, CA",
                        },
                        "format": {
                            "type": "string",
                            "enum": ["celsius", "fahrenheit"],
                            "description": "The temperature unit to use. Infer this from the users location.",
                        },
                    },
                    "required": ["location", "format"],
                },
            )
        )
    ],
    messages=[
        UserMessage(content="What's the weather like today in Paris?"),
        ],
)

tokens = tokenizer.encode_chat_completion(completion_request).tokens

out_tokens, _ = generate([tokens], model, max_tokens=64, temperature=0.0, eos_id=tokenizer.instruct_tokenizer.tokenizer.eos_id)
result = tokenizer.instruct_tokenizer.tokenizer.decode(out_tokens[0])

print(result)
```

### One-file-ref

If you want a self-contained implementation, look at `one_file_ref.py`, or run it with 

```
python -m one_file_ref $M7B_DIR
```

which should give something along the following lines:

```
This is a test of the emergency broadcast system. This is only a test.

If this were a real emergency, you would be told what to do.

This is a test
=====================
This is another test of the new blogging software. I’m not sure if I’m going to keep it or not. I’m not sure if I’m going to keep
=====================
This is a third test, mistral AI is very good at testing. 🙂

This is a third test, mistral AI is very good at testing. 🙂

This
=====================
```

**Note**: To run self-contained implementations, you need to do a local installation.

### Test

To run logits equivalence:
```
python -m pytest tests
```

## Deployment

The `deploy` folder contains code to build a [vLLM](https://M7B_DIR.com/vllm-project/vllm) image with the required dependencies to serve the Mistral AI model. In the image, the [transformers](https://github.com/huggingface/transformers/) library is used instead of the reference implementation. To build it:

```bash
docker build deploy --build-arg MAX_JOBS=8
```

Instructions to run the image can be found in the [official documentation](https://docs.mistral.ai/quickstart).


## Model platforms

- Use Mistral models on [Mistral AI official API](https://console.mistral.ai/) (La Plateforme)
- Use Mistral models via [cloud providers](https://docs.mistral.ai/deployment/cloud/overview/)

