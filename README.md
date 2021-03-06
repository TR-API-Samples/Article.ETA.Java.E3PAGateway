# Article.ETA.Java.E3PAGateway

The EU’s Markets in Financial Instruments MIFID II Directive, will introduce significant changes for investment firms and other financial markets participants when the EU member states implemented them in January 2018. This article discusses the capability for existing TREP clients to transmit pre-trade FIX messages to registered Approved Publication Arrangements (APA) to satisfy MIFID II regulatory requirements.

The E3PA Gateway connects to APAs via the FIX protocol using the QuickFIX/J engine, logs all FIX traffic, and supports a simple management interface.

In many cases, data subject to pre-trade reporting obligations is already available via TREP in the form of MarketPrice updates which are shared via OMM posts. The E3PA Gateway will act as an additional endpoint for OMM post messages, and convert them to FIX format using a simple transformation directive and forward them to the configured APA for processing.

More information is available from the [E3PAGateway article](https://developers.thomsonreuters.com/article/mifid-ii-enterprise-pre-post-trade-publication-apas-e3pa) on the [Thomson Reuters Developer Portal](https://developers.thomsonreuters.com).

## Preqrequisites

### TREP Requirements
E3PA is compatible with TREP versions 2 and 3. Messages are delivered to it with on-stream or off-stream OMM Post messages.

### E3PA Requirements
E3PA requires any machine running Java, with a minimum of 2GB of memory and Internet access. Gradle is also required to build E3PA and pull down dependencies from Maven. Docker is required to build and run the Docker image if you chose that option.

## Building 

### Building the project
Download the project which includes the E3PA Java source, build scripts and Thomson Reuters libraries on any machine running Java and Gradle.

In the directory containing the file `build.gradle`, run the command `gradle makeDist`. This will create a file named `apagateway_dist.tar.gz`. This file can then be moved to the location from where the E3PA gateway will be run.

### Building a Docker image
Also included is `Dockerfile` that can be used to create a Docker image. Run `docker build -t e3pa-gateway:latest .` from the directory with the Dockerfile after you have built the project with Gradle as outlined above. This will create a Docker image that can be run.

If you wish to run the image on a different machine, run `docker save -o e3pa-gateway.tar e3pa-gateway` to create a TAR file that can be moved. On the target machine run `docker load -i e3pa-gateway.tar` to create the image.

In the future you will also be able to download a Docker image from [DockerHub](https://hub.docker.com/).

### Project Directory Structure
* package/ -- root directory

  * app/ -- all software for the gateway   

  * client/omm/ -- FIX and UPA dictionaries

  * rules/ -- JSON rules file location

  * gateway.sh -- script to start E3PA 

  * utils.sh -- script used by gateway.sh 

## Running the E3PA gateway server
E3PA can be run from a shell script or can be run from a Docker image. Both methods are describe below.

To start the E3PA server extract the `apagateway_dist.tar.gz` file created after you ran the Gradle build and go to the `package` directory. Run the `gateway.sh` with the `--server` option to start the server. There are many command line options available, which are documented at the [E3PAGateway article](https://developers.thomsonreuters.com/articles-samples).

`gateway.sh --server <command-line options>`

A sample start command might look like:

`./gateway.sh --server -a rmds -i 256 --service AUTEX_APA -s AUTEX -t APA_DEST -e 198.100.10.92 -g 7000 -k FIXT.1.1 --fixdictionary ./client/omm/FDMFixFieldDictionary --fixenums ./client/omm/FDMenumtypes.def --postdictionary ./client/omm//RDMFieldDictionary --postenums ./client/omm/enumtype.def --rulesfile ./rules/rules.json --filelog ./logs --json ./logs`

### Custom RDM Dictionaries

We have included the RDM dictionaries, however if you have customized your own you can specify the location using the `--postdictionary` and `--postenums` options at startup.

### OMM Post ACK/NACK

If the message is successfully accepted and transformed to FIX an OMM ACK message will be returned. This does not mean that the APA successfully accepted the FIX message, only that it was converted. If the message was not converted a NACK will be returned.

### Logs

All outbound FIX messages to APAs and inbound from APAs are logged in standard FIX logs, along with all other FIX events.  All FIX messages are also logged in JSON formatted logs, which then can easily be loaded to a data storage application such as Elasticsearch. The FIX log directory and JSON log directory are each specified at startup using `--filelog` and `--json` options.

## Defining JSON Transformation Rules

In order to convert OMM Post messages to FIX, you must specify mapping logic in a JSON file. The rules will be applied to each incoming post message and applied to convert it to FIX to be sent to the APA destination.

We have included sample rules files that follow a few APA pre-trade FIX specs, but since the OMM Post dictionaries are highly customized by clients, each user will need to tweak the JSON transformation rules to capture the OMM Post fields.

For more information on writing JSON rules, please see the [E3PAGateway article](https://developers.thomsonreuters.com/articles-samples).

## JMX Management

The QuickFIX/J engine supports standard JMX commands for FIX engine management. You can use any JMX compatible application, including JConsole. The default JMX port for the E3PA gateway is 6998 and you can access it using JConsole by running `jconsole hostname:6998`. QuickFIX/J commands are all under the org.quickfix MBean. See http://www.quickfixj.org/quickfixj/usermanual/1.6.3/usage/jmx.html for more info.

## Running with Docker
The E3PA Gateway has been built and tested on Docker and will be distributed via DockerHub. Docker is a software container platform that allows us to ship the entire E3PA environment in one package that can be run on any Docker-enabled host. The environment contains the operating system, Java version, etc., any libraries and settings. For more info on Docker see https://www.docker.com/what-docker.



