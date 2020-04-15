---
uid: InstallOSIsoftAdapterForModbusTCPUsingDocker
---

# Install OSIsoft Adapter for Modbus TCP using Docker

Docker is a set of tools that you can use on Linux to manage application deployments.

**Note:** If you want to use Docker, you must be familiar with the underlying technology and have determined that it is appropriate for your planned use of the Modbus TCP adapter. Docker is not a requirement to use Modbus TCP adapter.

This topic provides examples of how to create a Docker container with the Modbus TCP adapter.

## Create a startup script for the adapter

1. Using any text editor, create a script similar to one of the following.

	**Note:** The script varies slightly by processor.

	<details>
	<summary>ARM32</summary>
	<pre>

		#!/bin/sh
		#local variables
		defaultPort=5590
		#regexp to only accept numerals
		re='^[0-9]+$'
			
		portConfigFile="/Modbus_linux-arm/appsettings.json"

		#validate the port number input
		if [ -z $portnum ] ; then
			portnum=${defaultPort}
			echo "Default value selected." ;
		else
			echo $portnum | grep -q -E $re
			isNum=$?
			if [ $isNum -ne 0 ] || [ $portnum -le 1023 ] || [ $portnum -gt 49151 ] ; then
				echo "Invalid input. Setting default value ${defaultPort} instead..."
				portnum=${defaultPort} ;
			fi
		fi

		echo "configuring port ${portnum}"
		#write out the port config file
		cat > ${portConfigFile} << EOF
		{
		"ApplicationSettings": {
			"Port": ${portnum},
			"ApplicationDataDirectory": "/usr/share/OSIsoft/Adapters/Modbus/Modbus"
			}
		}
		EOF
		exec /Modbus_linux-arm/OSIsoft.Data.System.Host
	</pre>
	</details>

	<details>
	<summary>ARM64</summary>
	<pre>

		#!/bin/sh
		#local variables
		defaultPort=5590
		#regexp to only accept numerals
		re='^[0-9]+$'
			
		portConfigFile="/Modbus_linux-arm64/appsettings.json"

		#validate the port number input
		if [ -z $portnum ] ; then
			portnum=${defaultPort}
			echo "Default value selected." ;
		else
			echo $portnum | grep -q -E $re
			isNum=$?
			if [ $isNum -ne 0 ] || [ $portnum -le 1023 ] || [ $portnum -gt 49151 ] ; then
				echo "Invalid input. Setting default value ${defaultPort} instead..."
				portnum=${defaultPort} ;
			fi
		fi

		echo "configuring port ${portnum}"
		#write out the port config file
		cat > ${portConfigFile} << EOF
		{
		"ApplicationSettings": {
			"Port": ${portnum},
			"ApplicationDataDirectory": "/usr/share/OSIsoft/Adapters/Modbus/Modbus"
			}
		}
		EOF
		exec /Modbus_linux-arm64/OSIsoft.Data.System.Host
	</pre>
	</details>

	<details>
	<summary>AMD64</summary>
	<pre>

		#!/bin/sh
		#local variables
		defaultPort=5590
		#regexp to only accept numerals
		re='^[0-9]+$'
			
		portConfigFile="/Modbus_linux-x64/appsettings.json"

		#validate the port number input
		if [ -z $portnum ] ; then
			portnum=${defaultPort}
			echo "Default value selected." ;
		else
			echo $portnum | grep -q -E $re
			isNum=$?
			if [ $isNum -ne 0 ] || [ $portnum -le 1023 ] || [ $portnum -gt 49151 ] ; then
				echo "Invalid input. Setting default value ${defaultPort} instead..."
				portnum=${defaultPort} ;
			fi
		fi

		echo "configuring port ${portnum}"
		#write out the port config file
		cat > ${portConfigFile} << EOF
		{
		"ApplicationSettings": {
			"Port": ${portnum},
			"ApplicationDataDirectory": "/usr/share/OSIsoft/Adapters/Modbus/Modbus"
			}
		}
		EOF
		exec /Modbus_linux-x64/OSIsoft.Data.System.Host
	</pre>
	</details>

2. Name the script *modbusdockerstart.sh* and save it to the directory where you plan to create the container.

## Create a Docker container containing the Modbus TCP adapter

1. Create the following Dockerfile in the directory where you want to create and run the container.

	**Note:** Dockerfile is the required name of the file. Use the variation according to your operating system:

	### ARM32

	```bash
	FROM ubuntu
	WORKDIR /
	RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends libicu60 libssl1.0.0
	COPY modbusdockerstart.sh /
	RUN chmod +x /modbusdockerstart.sh
	ADD ./Modbus_linux-arm.tar.gz .
	ENTRYPOINT ["/modbusdockerstart.sh"]
	```
	### ARM64

	```bash
	FROM ubuntu
	WORKDIR /
	RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends libicu60 libssl1.0.0
	COPY modbusdockerstart.sh /
	RUN chmod +x /modbusdockerstart.sh
	ADD ./Modbus_linux-arm64.tar.gz .
	ENTRYPOINT ["/modbusdockerstart.sh"]
	```

	### AMD64 (x64)

	```bash
	FROM ubuntu
	WORKDIR /
	RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends libicu60 libssl1.0.0
	COPY modbusdockerstart.sh /
	RUN chmod +x /modbusdockerstart.sh
	ADD ./Modbus_linux-x64.tar.gz .
	ENTRYPOINT ["/modbusdockerstart.sh"]
	```

2. Copy the *Modbus_linux-\<platform>.tar.gz* file to the same directory as the Dockerfile.

3. Copy the *modbusdockerstart.sh* script to the same directory.

4. Run the following command line in the same directory (sudo may be necessary):

	```bash
	docker build -t modbusadapter .
	```

## Run the Modbus TCP adapter Docker container

### REST access from the local host to the Docker container

Complete the following to run the container:

1. Use the docker container image modbusadapter created previously.
2. Type the following in the command line (sudo may be necessary):

	```bash
	docker run -d --network host modbusadapter
	```

Port 5590 is accessible from the host and you can make REST calls to Modbus TCP adapter from applications on the local host computer. In this example, all data stored by the Modbus TCP adapter is stored in the container itself. When the container is deleted, the data stored is also deleted.

### Provide persistent storage for the Docker container

Complete the following to run the container:

1. Use the docker container image modbusadapter created previously.
2. Type the following in the command line (sudo may be necessary):

	```bash
	docker run -d --network host -v /modbus:/usr/share/OSIsoft/ modbusadapter
	```

Port 5590 is accessible from the host and you can make REST calls to Modbus TCP adapter from applications on the local host computer. In this example, all data that is written to the container is instead written to the host directory. In this example, the host directory is a directory on the local machine, */modbus*. You can specify any directory.

### Port number change

To use a different port other than 5590 you can specify a **portnum** variable on the `docker run` command line. For example, to start the adapter using port 6000 instead of 5590, you use the command line:

```bash
docker run -d -e portnum=6000 --network host modbusadapter
```

Instead of accessing the REST API using port 5590 you use port 6000. The following curl command returns the configuration for the container.

```bash
curl http://localhost:6000/api/v1/configuration
```

### Remove REST access to the Docker container

If you remove the `--network host` option from the docker run command, no REST access is possible from outside the container. This may be of value where you want to host an application in the same container as Modbus TCP adapter, and do not want to have external REST access enabled.