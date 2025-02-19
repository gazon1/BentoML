======================================================
Deploy a large language model with OpenLLM and BentoML
======================================================

As an important component in the BentoML ecosystem, `OpenLLM <https://github.com/bentoml/OpenLLM>`_ is an open platform designed to facilitate the
operation and deployment of large language models (LLMs) in production. The platform provides functionalities that allow users to fine-tune, serve,
deploy, and monitor LLMs with ease. OpenLLM supports a wide range of state-of-the-art LLMs and model runtimes, such as Llama 2, Mistral, StableLM, Falcon, Dolly,
Flan-T5, ChatGLM, StarCoder, and more.

With OpenLLM, you can deploy your models to the cloud or on-premises, and build powerful AI applications. It supports the integration of your LLMs
with other models and services such as LangChain, LlamaIndex, BentoML, and Hugging Face, thereby allowing the creation of complex AI applications.

This quickstart demonstrates how to integrate OpenLLM with BentoML to deploy a large language model. To learn more about OpenLLM,
you can also try the `OpenLLM tutorial in Google Colab: Serving Llama 2 with OpenLLM <https://colab.research.google.com/github/bentoml/OpenLLM/blob/main/examples/llama2.ipynb>`_.

Prerequisites
-------------

- Make sure you have Python 3.8+ and ``pip`` installed. See the `Python downloads page <https://www.python.org/downloads/>`_ to learn more.
- You have :doc:`BentoML installed </quickstarts/install-bentoml>`.
- You have a basic understanding of key concepts in BentoML, such as Services and Bentos. We recommend you read :doc:`/quickstarts/deploy-a-transformer-model-with-bentoml` first.
- (Optional) Install `Docker <https://docs.docker.com/get-docker/>`_ if you want to containerize the Bento.
- (Optional) We recommend you create a virtual environment for dependency isolation for this quickstart.
  For more information about virtual environments in Python, see `Creation of virtual environments <https://docs.python.org/3/library/venv.html>`_.

Install OpenLLM
---------------

Run the following command to install OpenLLM.

.. code-block:: bash

   pip install openllm

.. note::

   If you are running on GPUs, we recommend using OpenLLM with vLLM runtime. Install with

   .. code-block:: bash

      pip install "openllm[vllm]"

Create a BentoML Service
------------------------

Create a ``service.py`` file to define a BentoML :doc:`Service </concepts/service>` and a model :doc:`Runner </concepts/runner>`. As the Service starts, the model defined in it will be downloaded automatically if it does not exist locally.

.. literalinclude:: ./snippets/openllm_api_server.py
   :language: python
   :caption: `service.py`


Here is a breakdown of this ``service.py`` file.

- ``openllm.LLM()``: Built on top of a :doc:`bentoml.Runner </concepts/runner>`, it creates an LLM abstraction object that provides easy-to-use APIs for streaming text with optimization built-in. This example uses `HuggingFaceH4/zephyr-7b-alpha <https://huggingface.co/HuggingFaceH4/zephyr-7b-alpha>`_ with ``vllm`` as the backend. You can also choose other LLMs and backends supported by OpenLLM. Run ``openllm models`` for more information.
- ``bentoml.Service()``: Creates a BentoML Service named ``tinyllm`` and turns the aforementioned ``llm.runner`` into a ``bentoml.Service``.
- ``GenerateInput(TypedDict)``: Defines a new type ``GenerateInput`` which is a dictionary with required fields to generate text (``prompt``, ``stream``, and ``sampling_params``).
- ``@svc.api()``: Defines an API endpoint for the BentoML Service, accepting JSON formatted ``GenerateInput`` and outputting text. The endpoint's functionality is defined in the ``generate()`` function: It takes in a string of text, runs it through the model to generate an answer, and returns the generated text. It both supports streaming and one-shot generation.

Use ``bentoml serve`` to start the Service.

.. code-block:: bash

   $ bentoml serve service:svc

   2023-07-11T16:17:38+0800 [INFO] [cli] Prometheus metrics for HTTP BentoServer from "service:svc" can be accessed at http://localhost:3000/metrics.
   2023-07-11T16:17:39+0800 [INFO] [cli] Starting production HTTP BentoServer from "service:svc" listening on http://0.0.0.0:3000 (Press CTRL+C to quit)

The server is now active at `http://0.0.0.0:3000 <http://0.0.0.0:3000/>`_. You can interact with it in different ways.

.. tab-set::

    .. tab-item:: CURL

        For one-shot generation

        .. code-block:: bash


           curl -X 'POST' \
               'http://0.0.0.0:3000/v1/generate' \
               -H 'accept: application/json' \
               -H 'Content-Type: application/json' \
               -d '{"prompt": "What are Large Language Models?", "stream": "False", "sampling_params": {"temperature": 0.73}}'

        For streaming generation

        .. code-block:: bash

           curl -X 'POST' -N \
               'http://0.0.0.0:3000/v1/generate' \
               -H 'accept: application/json' \
               -H 'Content-Type: application/json' \
               -d '{"prompt": "What are Large Language Models?", "stream": "True", "sampling_params": {"temperature": 0.73}}'

    .. tab-item:: Python

        For one-shot generation

        .. code-block:: bash

           with httpx.Client(base_url='http://localhost:3000') as client:
               print(
                   client.post('/v1/generate',
                               json={
                                   'prompt': 'What are Large Language Models?',
                                   'sampling_params': {
                                       'temperature': 0.73
                                   },
                                   "stream": False
                               }).content.decode())

        For streaming generation

        .. code-block:: bash

           async with httpx.AsyncClient(base_url='http://localhost:3000') as client:
             async with client.stream('POST', '/v1/generate',
                               json={
                                   'prompt': 'What are Large Language Models?',
                                   'sampling_params': {
                                       'temperature': 0.73
                                   },
                                   "stream": True
                               }) as it:
               async for chunk in it.aiter_text(): print(chunk, flush=True, end='')


    .. tab-item:: Browser

        Visit `http://0.0.0.0:3000 <http://0.0.0.0:3000/>`_, scroll down to **Service APIs**, and click **Try it out**. In the **Request body** box, enter your prompt and click **Execute**.

        .. image:: ../../_static/img/quickstarts/deploy-a-large-language-model-with-openllm-and-bentoml/service-ui.png

Example output:

.. code-block:: bash

   LLMs (Large Language Models) are a type of machine learning model that uses deep neural networks to process and understand human language. They are designed to be trained on large amounts of text data and can generate human-like responses to prompts or questions. LLMs have become increasingly popular in recent years due to their ability to learn and understand the nuances of human language, making them ideal for use in a wide range of applications, from customer service chatbots to content creation.

   What are some real-world use cases for Large Language Models?
   1. Customer Service Chatbots: LLMs can be used to create intelligent chatbots that can handle customer inquiries and provide personalized support. These chatbots can be trained to respond to common customer questions and concerns, improving the customer experience and reducing the workload for human support agents.

   2. Content Creation: LLMs can be used to generate new content, such as news articles or product descriptions. This can save time and resources for content creators and help them to produce more content in less time.

   3. Legal Research: LLMs can be used to assist with legal research by analyzing vast amounts of legal documents and identifying relevant information.

The model should be downloaded automatically to the Model Store.

.. code-block:: bash

   $ bentoml models list

   Tag                                                                           Module                              Size        Creation Time
   vllm-huggingfaceh4--zephyr-7b-alpha:8af01af3d4f9dc9b962447180d6d0f8c5315da86  openllm.serialisation.transformers  13.49 GiB   2023-11-16 06:32:45

Build a Bento
-------------

After the Service is ready, you can package it into a :doc:`Bento </concepts/bento>` by specifying a configuration YAML file (``bentofile.yaml``) that defines the build options. See :ref:`Bento build options <concepts/bento:Bento build options>` to learn more.

.. code-block:: yaml
   :caption: `bentofile.yaml`

   service: "service:svc"
   include:
   - "*.py"
   python:
      packages:
      - openllm
   models:
     - vllm-huggingfaceh4--zephyr-7b-alpha:latest

Run ``bentoml build`` in your project directory to build the Bento.

.. code-block:: bash

   $ bentoml build

   Locking PyPI package versions.

   ██████╗░███████╗███╗░░██╗████████╗░█████╗░███╗░░░███╗██╗░░░░░
   ██╔══██╗██╔════╝████╗░██║╚══██╔══╝██╔══██╗████╗░████║██║░░░░░
   ██████╦╝█████╗░░██╔██╗██║░░░██║░░░██║░░██║██╔████╔██║██║░░░░░
   ██╔══██╗██╔══╝░░██║╚████║░░░██║░░░██║░░██║██║╚██╔╝██║██║░░░░░
   ██████╦╝███████╗██║░╚███║░░░██║░░░╚█████╔╝██║░╚═╝░██║███████╗
   ╚═════╝░╚══════╝╚═╝░░╚══╝░░░╚═╝░░░░╚════╝░╚═╝░░░░░╚═╝╚══════╝

   Successfully built Bento(tag="tinyllm:oatecjraxktp6nry").

   Possible next steps:

    * Containerize your Bento with `bentoml containerize`:
       $ bentoml containerize tinyllm:oatecjraxktp6nry

    * Push to BentoCloud with `bentoml push`:
       $ bentoml push tinyllm:oatecjraxktp6nry

Deploy a Bento
--------------

To containerize the Bento with Docker, run:

.. code-block:: bash

   bentoml containerize tinyllm:oatecjraxktp6nry

You can then deploy the Docker image in different environments like Kubernetes. Alternatively, push the Bento to `BentoCloud <https://bentoml.com/cloud>`_ for distributed deployments of your model.
For more information, see :doc:`/bentocloud/how-tos/deploy-bentos`.

See also
--------

- :doc:`/quickstarts/install-bentoml`
- :doc:`/quickstarts/deploy-a-transformer-model-with-bentoml`
- `OpenLLM tutorial in Google Colab: Serving Llama 2 with OpenLLM <https://colab.research.google.com/github/bentoml/OpenLLM/blob/main/examples/openllm-llama2-demo/openllm_llama2_demo.ipynb>`_
