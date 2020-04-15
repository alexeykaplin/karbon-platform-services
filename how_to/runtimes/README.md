## Runtime Environments

A runtime environment is a command execution environment to run applications written in a particular language or associated with a specific Docker registry or file. Each Function added to a Data Pipeline is executed via its own specified Runtime Environment.

Xi IoT includes standard runtime environments including but not limited to the following. These runtimes are read-only and cannot be edited, updated, or deleted by users. They are available to all projects, functions, and associated container registries.

* **Golang**
* **NodeJS**
	* Node.js functions can be run in context of a data pipeline. A transformation function must accept context as well as message payload as parameters. Context can be used to query function parameters passed in when function has been instantiated. Moreover context is used to send messages to next stage in data pipeline.
	* Following is a basic Node.js function template:


```javascript
function main(ctx, msg) {
   return new Promise(function(resolve, reject) {
      // log list of transformation parameters
      console.log("Config", ctx.config)
      // log length of message payload
      console.log(msg.length)
      // forward message to next stage in pipeline
      ctx.send(msg)
      // complete promise
      resolve()
   })
}
exports.main = main
```

All functions must export main which returns a promise.

Expected output:

```console
Config { IntParam: '42', StringParam: 'hello' }
2764855
```

**Note**
Packages available in NodeJS Runtime
	* alpine-baselayout
	* alpine-keys
	* apk-tools
	* busybox
	* libc-utils
	* libgcc
	* libressl2.5-libcrypto
	* libressl2.5-libssl
	* libressl2.5-libtls
	* libstdc++
	* musl
	* musl-utils
	* scanelf
	* ssl_client
	* zlib

* **Python**
	* Functions can be executed in data pipelines to transform and filter data. Transformations are functions used to process single messages and optionally forward them to next stage in data pipeline. The next stage could be another transformation or destination of the data pipeline on edge or in the cloud. Transformation can accept parameters. In Python parameters are passed as dictionary to transformation. The following script demonstrates some basic concepts:
```python
import logging
# Python function are invoked with context and message payload.
# The context can be used to retrieve metadata about the message and allows
# function to send mesagges to next stage in stream. In this sample we just
# log message payload and forward it as is to next stage.
def main(ctx, msg):
      logging.info("Parameters: %s", ctx.get_config())
      logging.info("Process %d bytes from %s at %s", msg, ctx.get_topic(), ctx.get_timestamp())
      # Forward to next stage in pipeline.
      ctx.send(msg)
```

Pass two parameters to the function:
* MyStringParam like the name suggests is a parameter of type string.
* MyIntParam is a number.

The function would produce the following console output when processing images from a camera:
```console
[2019-03-12 04:57:26,820 root INFO] Parameters: {u'MyIntParam':u'42', u'MyStringParam': u'hello'}
[2019-03-12 04:57:26,820 root INFO] Process 2764855 bytes from rtsp://184.72.239.149:554/vod/mp4:BigBuckBunny_175k.mov at 1552366646754939017
```

**Methods provided by ctx**

* **ctx.get_config()** - returns a dict of parameters passed to the function.
* **ctx.get_topic()** - returns the topic (string) on which the current message was received. In this case, it is the topic is set to RTSP topic from which image has been received.
* **ctx.get_timestamp()** - returns the time in nanoseconds since epoch (Jan 1st, 1970 UTC).
* **ctx.send()** - Takes bytes as input and forwards it to the next stage in the pipeline. If the input is not of type bytes, an error is thrown and a corresponding alert is raised in Xi IoT.

**In memory caching**
```python
import logging

counter=0

def main(ctx, msg):
      global counter
      logging.info("This is message number %d", counter)
      counter+=1
      # Forward to next stage in pipeline.
      ctx.send(msg)
```

Script produces following output:
```console
[2019-03-12 05:19:04,844 root INFO] This is message number 0
[2019-03-12 05:19:05,846 root INFO] This is message number 1
[2019-03-12 05:19:06,836 root INFO] This is message number 2
[2019-03-12 05:19:07,837 root INFO] This is message number 3
[2019-03-12 05:19:08,838 root INFO] This is message number 4
```

The data pipeline has been configured to sample every second.

Transformations are not limited to just filter or pass thru messages. A transformation can send as many messages to the next stage in pipeline as required by using context:

```python
import logging

# Transformation can send more messages than they receive.
def main(ctx, msg):
      logging.info("Process %d bytes from %s at %s", len(msg), ctx.get_topic(), ctx.get_timestamp())
      m = len(msg) / 2
      # split message in two halves
      ctx.send(msg[:m])
      ctx.send(msg[m:])
```	

Logs will reflect how message have been split:
```console
[19-03-12 05:30:51,696 root INFO] Process 2764855 bytes from rtsp://184.72.239.149:554/vod/mp4:BigBuckBunny_175k.mov
[2019-03-12 05:30:51,697 root INFO] Send 1382427 bytes
[2019-03-12 05:30:51,697 root INFO] Send 1382428 
```

**Note**

Packages available in Python 2 Runtime
* backports-abc 0.5
* elasticsearch 6.3.1
* elasticsearch-dsl 6.3.1
* futures 3.2.0
* ipaddress 1.0.22
* kafka-python 1.4.4
* msgpack 0.5.6
* nats-client 0.8.2
* paho-mqtt 1.4.0
* pip 18.1
* prometheus-client 0.5.0
* protobuf 3.6.1
* python-dateutil 2.7.5
* setuptools 40.6.3
* singledispatch 3.4.0.3
* six 1.12.0
* tornado 5.1.1
* urllib3 1.24.1
* virtualenv 16.2.0
* wheel 0.32.3
* requests 2.20.1

Packages available in Python 3 Runtime
* asyncio-nats-client 0.8.2
* elasticsearch 6.3.1
* elasticsearch-dsl 6.3.1
* kafka-python 1.4.4
* msgpack 0.5.6
* paho-mqtt 1.4.0
* pip 18.1
* prometheus-client 0.5.0
* protobuf 3.6.1
* python-dateutil 2.7.5
* setuptools 40.6.3
* six 1.12.0
* urllib3 1.24.1
* wheel 0.32.3
* requests 2.20.1

Packages available in Tensorflow Python 2 Runtime
* Absl-py 0.1.10
* astor 0.6.2
* astroid 1.6.1
* backports-abc 0.5
* backports.functools-lru-cache 1.5
* backports.shutil-get-terminal-size 1.0.0
* backports.weakref 1.0.post1
* bleach 1.5.0
* configparser 3.5.0
* cycler 0.10.0
* decorator 4.2.1
* elasticsearch 6.3.1
* elasticsearch-dsl 6.3.1
* entrypoints 0.2.3
* enum34 1.1.6
* funcsigs 1.0.2
* functools32 3.2.3.post2
* futures 3.2.0
* gast 0.2.0
* grpcio 1.10.0
* h5py 2.7.1
* html5lib 0.9999999
* imutils 0.4.5
* ipaddress 1.0.22
* ipykernel 4.8.2
* ipython 5.5.0
* ipython-genutils 0.2.0
* ipywidgets 7.1.2
* isort 4.3.3
* Jinja2 2.10
* jsonschema 2.6.0
* jupyter 1.0.0
* jupyter-client 5.2.3
* jupyter-console 5.2.0
* jupyter-core 4.4.0
* kafka-python 1.4.4
* kiwisolver 1.0.1
* lazy-object-proxy 1.3.1
* Markdown 2.6.11
* MarkupSafe 1.0
* matplotlib 2.2.2
* mccabe 0.6.1
* mistune 0.8.3
* mock 2.0.0
* msgpack 0.5.6
* nats-client 0.8.2
* nbconvert 5.3.1
* nbformat 4.4.0
* notebook 5.4.1
* numpy 1.14.0
* opencv-python 3.4.0.12
* paho-mqtt 1.4.0
* pandas 0.22.0
* pandocfilters 1.4.2
* pathlib2 2.3.0
* pbr 4.0.0
* pexpect 4.4.0
* pickleshare 0.7.4
* Pillow 5.0.0
* pip 18.1
* prometheus-client 0.5.0
* prompt-toolkit 1.0.15
* protobuf 3.5.1
* ptyprocess 0.5.2
* Pygments 2.2.0
* pylint 1.8.2
* pyparsing 2.2.0
* python-dateutil 2.7.5
* pytz 2018.3
* pyzmq 17.0.0
* qtconsole 4.3.1
* scandir 1.7
* scikit-learn 0.19.1
* scipy 1.0.0
* Send2Trash 1.5.0
* setuptools 40.6.3
* simplegeneric 0.8.1
* singledispatch 3.4.0.3
* six 1.11.0
* sklearn 0.0
* subprocess32 3.2.7
* tensorboard 1.7.0
* tensorflow 1.7.0
* termcolor 1.1.0
* terminado 0.8.1
* testpath 0.3.1
* tornado 5.1.1
* traitlets 4.3.2
* urllib3 1.24.1
* virtualenv 16.2.0
* wcwidth 0.1.7
* webencodings 0.5.1
* Werkzeug 0.14.1
* wheel 0.32.3
* widgetsnbextension 3.1.4
* wrapt 1.10.11

### Build a Custom Runtime Environment

You may need a custom runtime for some third party packages or OS distributions (like Linux) which might have dependencies not covered with the built-in Xi IoT runtimes.

Like the built-in runtime environments, custom runtimes are docker images that can run functions. **A runtime container image must include the Xi IoT language-specific runtime bundle.**

The bundle’s runtime environment is responsible for:

* Bootstraping the container by downloading the script assigned to that container at runtime
* Receiving messages and events
* Providing the API necessary to inspect and forward messages
* Reporting statistics and alerts to Xi IoT control plane

Nutanix provides custom runtime support for three languages:
* [**Python 2**](https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/python2-env.tgz)
* [**Python 3**](https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/python-env.tgz)
* [**NodeJS**](https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/node-env.tgz)

Python 2 is distinguished from Python 3, as Python 3 syntax and libraries are not backward-compatible.

**Note**
Custom Golang runtime environments are not supported. Use the provided standard Golang runtime environment in this case.

This sample Dockerfile builds a custom runtime environment able to run Python 3 functions:
```
FROM python:3.6
RUN python -V
# Check Python version
RUN python -c 'import sys; sys.exit(sys.version_info.major != 3)'
# We need Python runtime environment to execute Python functions.
RUN wget https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/python-env.tgz
RUN tar xf /python-env.tgz
# Bundle does not come with all required packages but defines them as PIP dependencies
RUN pip install -r /python-env/requirements.txt
# In this example we install Kafka client for Python as additional 3rd party software
RUN pip install kafka-python

# Containers should NOT run as root as a good practice
# We mandate all runtime containers to run as user 10001
USER 10001
# Finally run Python function worker which pull and executes functions.
CMD ["/python-env/run.sh"]
```

Build this container as usual by invoking “docker build”:
```console
$ docker build -t edgecomputing/sample-env -f Dockerfile .
Sending build context to Docker daemon   2.56kB
Step 1/9 : FROM python:3.6
Step 2/9 : RUN python -V
Step 3/9 : RUN python -c 'import sys; sys.exit(sys.version_info.major != 3)'
Step 4/9 : RUN wget https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/python-env.tgz
Step 5/9 : RUN tar xf /python-env.tgz
Step 6/9 : RUN pip install -r /python-env/requirements.txt
Step 7/9 : RUN pip install kafka-python
Step 8/9 : USER 10001
Step 9/9 : CMD ["/python-env/run.sh"]
Removing intermediate container 52d45f3db900
---> 95a878cde355
Successfully built 95a878cde355
Successfully tagged edgecomputing/sample-env:latest
```

Upload the docker image to a container registry:
```console
$ docker tag edgecomputing/sample-env:latest $DOCKER_REPO/sample-env:v1.1
$ docker push $DOCKER_REPO/sample-env:v1.1
```

**Note**
Xi IoT edges pull runtime images using an ‘IfNotPresent’ policy. To ensure updates are pulled, tag your container using a specific version and increment it on updates rather than relying on the ‘latest’ tag.

**Note**
Docker Hub, AWS Elastic Container Registry, and GCP Container Registry registries are supported.

#### Example Custom Runtimes

* NodeJS
```
FROM node:9

RUN wget https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/node-env.tgz
RUN tar xf /node-env.tgz

WORKDIR /node-env
RUN npm install
# Containers should NOT run as root as a good practice
USER 10001
CMD ["/node-env/run.sh"]
```

* Python 3
```
FROM python:3.6

RUN python -V
# Check Python version
RUN python -c 'import sys; sys.exit(sys.version_info.major != 3)'
# We need Python runtime environment to execute Python functions.
RUN wget https://s3-us-west-2.amazonaws.com/ntnxsherlock-runtimes/python-env.tgz
RUN tar xf /python-env.tgz
# Bundle does not come with all required packages but defines them as PIP dependencies
RUN pip install -r /python-env/requirements.txt
# In this example we install Kafka client for Python as additional 3rd party software
RUN pip install kafka-python

# Containers should NOT run as root as a good practice
# We mandate all runtime containers to run as user 10001
USER 10001
# Finally run Python function worker which pull and executes functions.
CMD ["/python-env/run.sh"]
```

Complete examples of creating custom runtimes:
* [**Python 2**](https://github.com/sharvil-kekre/xi-iot/tree/sharvil/how_to/realtime_data_pipeline/python2)
* [**Python 3**](https://github.com/sharvil-kekre/xi-iot/tree/sharvil/how_to/realtime_data_pipeline/python3)
* [**NodeJS**](https://github.com/sharvil-kekre/xi-iot/tree/sharvil/how_to/realtime_data_pipeline/nodejs)

### Creating a Runtime Environment¶
View this topic in the [Xi IoT Infrastructure Admin Guide](https://portal.nutanix.com/page/documents/details/?targetId=Xi-IoT-Infra-Admin-Guide:edg-iot-runtime-create-t.html), available from the Nutanix Support Portal.

### Editing a Runtime Environment¶
View this topic in the [Xi IoT Infrastructure Admin Guide](https://portal.nutanix.com/page/documents/details/?targetId=Xi-IoT-Infra-Admin-Guide:edg-iot-runtime-create-t.html), available from the Nutanix Support Portal.

### Removing a Runtime Environment¶
View this topic in the [Xi IoT Infrastructure Admin Guide](https://portal.nutanix.com/page/documents/details/?targetId=Xi-IoT-Infra-Admin-Guide:edg-iot-runtime-create-t.html), available from the Nutanix Support Portal.