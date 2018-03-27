# Azure IoT Edge 101 

Getting Started with Azure IoT Edge v1. 

## Demo Steps

These steps are taken generally, with some variation, from [Quickstart: Deploy your first IoT Edge module from the Azure portal to a Windows device - preview](https://docs.microsoft.com/en-us/azure/iot-edge/quickstart)

### Ensure Pre-reqs:

1. You will need:

  - Docker
  - .NET Core 2.0
  - Python 2.7
  - The Azure IoT Edge Runtime Control script:

    ```bash
    pip install -U azure-iot-edge-runtime-ctl
    ```

### Create the Azure IoT Hub

1. Run 

    ```bash
    az group create -g edge101group -l westus
    ```

2. Then

    ```bash
    az iot hub create -n edge101hub -g edge101group --sku S1
    ```

### Create the Azure IoT Edge Device ID

1. Go to the [portal](https://portal.azure.com) and open the `edge101hub` you just created

1. On the left edge of the hub's blade, click on "**IoT Edge (preview)**", then on the top of that blade, click "**+ Add IoT Edge Device**" 

1. Give the new edge device an id, (`edge101edge`), leave all other fields at the defaults, and click "**Save**"

1. Once the new edge device ID is created, click on it's name, and copy it's "**Connection String - Primary**".  Save it for future use:

    ```text
    HostName=edge101hub.azure-devices.net;DeviceId=edge101edge;SharedAccessKey=qFo1k8DqPTQ0SL/j3zIW/Ma89WFaY2QZBw+Q2TQXYYw=
    ```

### Configure the IoT Edge instance

1. Run the following, passing in the connection string for the edge device you created previously:

    ```bash
    iotedgectl setup --connection-string "{device connection string}" --nopass
    ```

    For example:

    ```bash
    iotedgectl setup --connection-string "HostName=edge101hub.azure-devices.net;DeviceId=edge101edge;SharedAccessKey=qFo1k8DqPTQ0SL/j3zIW/Ma89WFaY2QZBw+Q2TQXYYw=" --nopass
    ```

1. Start the runtime

    ```bash
    iotedgectl start
    ```

1. And verify that the `edgeAgent` container is running:

    ```bash
    docker ps
    ```

    With output similar to:

    ```bash
    CONTAINER ID  IMAGE                                     ...  NAMES
    27cf5eefdcbf  microsoft/azureiotedge-agent:1.0-preview  ...  edgeAgent
    ``` 

1. Notice that there is only one module (container) running, `edgeAgent`. The `edgeHub` is NOT running because no modules have been specified so the `edgeHub` is not needed to broker the communication between them. 

1. View the logs fromt the `edgeAgent` module

    > **Note**: Keep the `edgeAgent` logs running in this terminal, and open another terminal for subsequent steps. That way we can monitor what is going on with the `edgeAgent` when we deploy the `tempSensor` module in the next step.

    ```bash
    docker logs -f edgeAgent
    ```

    With output similar to:

    ```bash
    2018-03-26 22:19:02.136 +00:00 [INF] - Starting module management agent.
    2018-03-26 22:19:02.336 +00:00 [INF] - Version - 1.0.0-preview021.10543704 (c87b52c93b13f03bf34da8d9ae650d55368ccecb)
    2018-03-26 22:19:02.337 +00:00 [INF] -
            █████╗ ███████╗██╗   ██╗██████╗ ███████╗
          ██╔══██╗╚══███╔╝██║   ██║██╔══██╗██╔════╝
          ███████║  ███╔╝ ██║   ██║██████╔╝█████╗
          ██╔══██║ ███╔╝  ██║   ██║██╔══██╗██╔══╝
          ██║  ██║███████╗╚██████╔╝██║  ██║███████╗
          ╚═╝  ╚═╝╚══════╝ ╚═════╝ ╚═╝  ╚═╝╚══════╝

    ██╗ ██████╗ ████████╗    ███████╗██████╗  ██████╗ ███████╗
    ██║██╔═══██╗╚══██╔══╝    ██╔════╝██╔══██╗██╔════╝ ██╔════╝
    ██║██║   ██║   ██║       █████╗  ██║  ██║██║  ███╗█████╗
    ██║██║   ██║   ██║       ██╔══╝  ██║  ██║██║   ██║██╔══╝
    ██║╚██████╔╝   ██║       ███████╗██████╔╝╚██████╔╝███████╗
    ╚═╝ ╚═════╝    ╚═╝       ╚══════╝╚═════╝  ╚═════╝ ╚══════╝

    2018-03-26 22:19:02.454 +00:00 [INF] - Edge agent attempting to connect to IoT Hub via AMQP...
    2018-03-26 22:19:03.287 +00:00 [INF] - Created persistent store at /tmp/edgeAgent
    2018-03-26 22:19:04.909 +00:00 [INF] - Edge agent connected to IoT Hub via AMQP.
    2018-03-26 22:19:05.377 +00:00 [INF] - Deployment config in edge agent's desired properties is empty.
    2018-03-26 22:19:05.391 +00:00 [ERR] - Error refreshing edge agent configuration from twin.
    Microsoft.Azure.Devices.Edge.Agent.Core.ConfigSources.ConfigEmptyException: This device has an empty configuration for the edge agent. Please set a deployment manifest.
      at Microsoft.Azure.Devices.Edge.Agent.IoTHub.EdgeAgentConnection.UpdateDeploymentConfig() in /opt/vsts/work/1/s/edge-agent/src/Microsoft.Azure.Devices.Edge.Agent.IoTHub/E
    dgeAgentConnection.cs:line 171
      at Microsoft.Azure.Devices.Edge.Agent.IoTHub.EdgeAgentConnection.<RefreshTwinAsync>d__20.MoveNext() in /opt/vsts/work/1/s/edge-agent/src/Microsoft.Azure.Devices.Edge.Agen
    t.IoTHub/EdgeAgentConnection.cs:line 124
    2018-03-26 22:19:06.065 +00:00 [INF] - Updated reported properties    
    ```

### Deploy the tempSensor module

1. Open the [Portal](https://portal.azure.com), open the `edgeiothub` IoT Hub, click "**IoT Edge (preview)**", select the `edge101edge` device, and finally click "**Set Modules**" along the top. 

1. On the "**Add Modules**" tab, click "**Add IoT Edge Module**".  

1. In the "**IoT Edge Modules**" tab

    - **Name**: `tempSensor`
    - **Image URI**: `microsoft/azureiotedge-simulated-temperature-sensor:1.0-preview`
    - **Click "Save"** - Leaving all other settings at default values

1. Back in "**Add Modules**", click "**Next**".

1. On the "**Specify Routes**" page, accept the default routes and click "**Next**":

    ```json
    {
      "routes": {
        "route": "FROM /* INTO $upstream"
      }
    }
    ```

1. On the "**Review Template**" page, review the template, and then click "**Submit**"

    ```json
    {
        "moduleContent": {
            "$edgeAgent": {
                "properties.desired": {
                    "schemaVersion": "1.0",
                    "runtime": {
                        "type": "docker",
                        "settings": {
                            "minDockerVersion": "v1.25",
                            "loggingOptions": ""
                        }
                    },
                    "systemModules": {
                        "edgeAgent": {
                            "type": "docker",
                            "settings": {
                                "image": "microsoft/azureiotedge-agent:1.0-preview",
                                "createOptions": "{}"
                            }
                        },
                        "edgeHub": {
                            "type": "docker",
                            "status": "running",
                            "restartPolicy": "always",
                            "settings": {
                                "image": "microsoft/azureiotedge-hub:1.0-preview",
                                "createOptions": "{}"
                            }
                        }
                    },
                    "modules": {
                        "tempSensor": {
                            "version": "1.0",
                            "type": "docker",
                            "status": "running",
                            "restartPolicy": "always",
                            "settings": {
                                "image": "microsoft/azureiotedge-simulated-temperature-sensor:1.0-preview",
                                "createOptions": "{}"
                            }
                        }
                    }
                }
            },
            "$edgeHub": {
                "properties.desired": {
                    "schemaVersion": "1.0",
                    "routes": {
                        "route": "FROM /* INTO $upstream"
                    },
                    "storeAndForwardConfiguration": {
                        "timeToLiveSecs": 7200
                    }
                }
            }
        }
    }    
    ```

1. Right after you click "**Submit**" above, immediately switch to the window where the logs for the `edgeAgent` are being shown and review what happens there.

    > **Note**: If the `edgeAgent` doesn't seem to be responding, you may need to restart the edge gateway with `iotedgctl restart`

    ```bash
    2018-03-27 11:46:45.523 +00:00 [INF] - Plan execution started for deployment 2
    2018-03-27 11:46:45.592 +00:00 [INF] - Executing command: "Command Group: (
      [docker pull microsoft/azureiotedge-hub:1.0-preview]
      [docker create --name edgeHub microsoft/azureiotedge-hub:1.0-preview]
      [docker start edgeHub]
    )"
    2018-03-27 11:46:45.601 +00:00 [INF] - Executing command: "docker pull microsoft/azureiotedge-hub:1.0-preview"
    2018-03-27 11:46:47.500 +00:00 [INF] - Executing command: "docker create --name edgeHub microsoft/azureiotedge-hub:1.0-preview"
    2018-03-27 11:46:47.631 +00:00 [INF] - Executing command: "docker start edgeHub"
    2018-03-27 11:46:48.799 +00:00 [INF] - Executing command: "Command Group: (
      [docker pull microsoft/azureiotedge-simulated-temperature-sensor:1.0-preview]
      [docker create --name tempSensor microsoft/azureiotedge-simulated-temperature-sensor:1.0-preview]
      [docker start tempSensor]
    )"
    2018-03-27 11:46:48.799 +00:00 [INF] - Executing command: "docker pull microsoft/azureiotedge-simulated-temperature-sensor:1.0-preview"
    2018-03-27 11:46:50.259 +00:00 [INF] - Executing command: "docker create --name tempSensor microsoft/azureiotedge-simulated-temperature-sensor:1.0-preview"
    2018-03-27 11:46:50.373 +00:00 [INF] - Executing command: "docker start tempSensor"
    2018-03-27 11:46:51.442 +00:00 [INF] - Plan execution ended for deployment 2
    2018-03-27 11:46:52.106 +00:00 [INF] - Updated reported properties    
    ```

1. In a new terminal window, run:

    ```bash
    docker ps
    ```

    And notice there are now two new modules (containers), `edgeHub` and `tempSensor` along with the original `edgeAgent`

    ```bash
    CONTAINER ID  IMAGE                                                            ...  NAMES
    a88d5b8e5920  microsoft/azureiotedge-simulated-temperature-sensor:1.0-preview  ...  tempSensor
    98b439b51d29  microsoft/azureiotedge-hub:1.0-preview                           ...  edgeHub
    27cf5eefdcbf  microsoft/azureiotedge-agent:1.0-preview                         ...  edgeAgent
    ```

1. In the same terminal window, run:

    ```bash
    docker logs -f edgeHub
    ```

    And review the output:

    ```bash
    Edge Hub Server Certificate File: /mnt/edgehub/edge-hub-server.cert.pfx
    Edge Hub CA Server Certificate File: /mnt/edgehub/edge-chain-ca.cert.pem
    SSL_CERTIFICATE_PATH=/mnt/edgehub
    SSL_CERTIFICATE_NAME=edge-hub-server.cert.pfx
    Executing: cp /mnt/edgehub/edge-chain-ca.cert.pem /usr/local/share/ca-certificates/edge-chain-ca.crt
    Executing: update-ca-certificates
    Updating certificates in /etc/ssl/certs...
    1 added, 0 removed; done.
    Running hooks in /etc/ca-certificates/update.d...
    done.
    Certificates installed successfully!
    2018-03-27 11:46:53.053 +00:00 [INF] - User profile is available. Using '"/home/edgehubuser/.aspnet/DataProtection-Keys"' as key repository; keys will not be encrypted at re
    st.
    2018-03-27 11:46:53.107 +00:00 [INF] - Creating key {d57d7fe4-8b07-40a7-b95f-bf0a69fa7bac} with creation date 2018-03-27 11:46:53Z, activation date 2018-03-27 11:46:53Z, and
    expiration date 2018-06-25 11:46:53Z.
    2018-03-27 11:46:53.123 +00:00 [WRN] - No XML encryptor configured. Key {d57d7fe4-8b07-40a7-b95f-bf0a69fa7bac} may be persisted to storage in unencrypted form.
    2018-03-27 11:46:53.130 +00:00 [INF] - Writing data to file '"/home/edgehubuser/.aspnet/DataProtection-Keys/key-d57d7fe4-8b07-40a7-b95f-bf0a69fa7bac.xml"'.
    2018-03-27 11:46:53.520 +00:00 [INF] - Starting Edge Hub.
    2018-03-27 11:46:53.522 +00:00 [INF] - Version - 1.0.0-preview021.10543704 (c87b52c93b13f03bf34da8d9ae650d55368ccecb)
    2018-03-27 11:46:53.522 +00:00 [INF] -
            █████╗ ███████╗██╗   ██╗██████╗ ███████╗
          ██╔══██╗╚══███╔╝██║   ██║██╔══██╗██╔════╝
          ███████║  ███╔╝ ██║   ██║██████╔╝█████╗
          ██╔══██║ ███╔╝  ██║   ██║██╔══██╗██╔══╝
          ██║  ██║███████╗╚██████╔╝██║  ██║███████╗
          ╚═╝  ╚═╝╚══════╝ ╚═════╝ ╚═╝  ╚═╝╚══════╝

    ██╗ ██████╗ ████████╗    ███████╗██████╗  ██████╗ ███████╗
    ██║██╔═══██╗╚══██╔══╝    ██╔════╝██╔══██╗██╔════╝ ██╔════╝
    ██║██║   ██║   ██║       █████╗  ██║  ██║██║  ███╗█████╗
    ██║██║   ██║   ██║       ██╔══╝  ██║  ██║██║   ██║██╔══╝
    ██║╚██████╔╝   ██║       ███████╗██████╔╝╚██████╔╝███████╗
    ╚═╝ ╚═════╝    ╚═╝       ╚══════╝╚═════╝  ╚═════╝ ╚══════╝

    2018-03-27 11:46:53.525 +00:00 [INF] - Initializing configuration
    2018-03-27 11:46:54.015 +00:00 [INF] - Created persistent store at /tmp/edgeHub
    2018-03-27 11:46:54.091 +00:00 [INF] - Attempting to connect to IoT Hub for client edge101edge/$edgeHub via AMQP...
    2018-03-27 11:46:55.876 +00:00 [INF] - Connected to IoT Hub for client edge101edge/$edgeHub via AMQP, with client operation timeout 60000.
    2018-03-27 11:46:55.889 +00:00 [INF] - Created cloud connection for client edge101edge/$edgeHub
    2018-03-27 11:46:55.918 +00:00 [INF] - New device connection for device edge101edge/$edgeHub
    2018-03-27 11:46:56.664 +00:00 [INF] - Created new message store
    2018-03-27 11:46:56.681 +00:00 [INF] - Started task to cleanup processed and stale messages
    2018-03-27 11:46:58.757 +00:00 [INF] - Obtained edge hub config from module twin
    2018-03-27 11:46:58.944 +00:00 [INF] - Set the following 1 route(s) in edge hub
    2018-03-27 11:46:58.946 +00:00 [INF] - route: FROM /* INTO $upstream
    2018-03-27 11:46:58.948 +00:00 [INF] - Updated message store TTL to 7200 seconds
    2018-03-27 11:46:58.949 +00:00 [INF] - Updated the edge hub store and forward configuration
    2018-03-27 11:46:58.949 +00:00 [INF] - Initialized edge hub configuration
    2018-03-27 11:46:58.949 +00:00 [INF] - Starting Http Server
    2018-03-27 11:46:59.783 +00:00 [INF] - Starting MQTT Server
    2018-03-27 11:46:59.834 +00:00 [INF] - Starting
    2018-03-27 11:46:59.945 +00:00 [INF] - Initializing TLS endpoint on port 8883.
    2018-03-27 11:47:00.053 +00:00 [INF] - Started
    2018-03-27 11:47:00.847 +00:00 [INF] - Attempting to connect to IoT Hub for client edge101edge/tempSensor via AMQP...
    2018-03-27 11:47:00.871 +00:00 [INF] - New token requested by client edge101edge/tempSensor, but using existing token as it is usable.
    2018-03-27 11:47:01.614 +00:00 [INF] - Connected to IoT Hub for client edge101edge/tempSensor via AMQP, with client operation timeout 60000.
    2018-03-27 11:47:01.619 +00:00 [INF] - No session state found in store for edge101edge/tempSensor
    2018-03-27 11:47:01.619 +00:00 [INF] - Created cloud connection for client edge101edge/tempSensor
    2018-03-27 11:47:01.620 +00:00 [INF] - New cloud connection created for device edge101edge/tempSensor
    2018-03-27 11:47:01.622 +00:00 [INF] - Successfully authenticated device edge101edge/tempSensor
    2018-03-27 11:47:01.623 +00:00 [INF] - Successfully generated identity for clientId edge101edge/tempSensor and username bssbp2/edge101edge/tempSensor/api-version=2017-06-30&
    DeviceClientType=Microsoft.Azure.Devices.Client%2F1.17.0-preview-001%20%28.NET%20Core%204.6.0.0%3B%20Linux%204.9.87-linuxkit-aufs%20%231%20SMP%20Wed%20Mar%2014%2015%3A12%3A1
    6%20UTC%202018%3B%20X64%29
    2018-03-27 11:47:01.627 +00:00 [INF] - ClientAuthenticated, edge101edge/tempSensor, 2561e1c0
    2018-03-27 11:47:01.668 +00:00 [INF] - New device connection for device edge101edge/tempSensor
    2018-03-27 11:47:01.715 +00:00 [INF] - Bind device proxy for device edge101edge/tempSensor
    2018-03-27 11:47:01.717 +00:00 [INF] - Binding message channel for device Id edge101edge/tempSensor
    2018-03-27 11:47:01.787 +00:00 [INF] - Set subscriptions from session state for edge101edge/tempSensor
    ```

1. In another new terminal window, run:

    ```bash
    docker logs -f tempSensor
    ```

    And review the output:

    ```bash
    Added Cert: /mnt/edgemodule/edge-device-ca.cert.pem
            03/27/2018 11:47:02> Sending message: 1, Body: [{"machine":{"temperature":21.035142293332676,"pressure":1.0040035524049884},"ambient":{"temperature":21.0663309314131
    41,"humidity":25},"timeCreated":"2018-03-27T11:47:02.108829Z"}]
            03/27/2018 11:47:07> Sending message: 2, Body: [{"machine":{"temperature":20.863129931671139,"pressure":0.98440720740557286},"ambient":{"temperature":20.851026331703
    657,"humidity":26},"timeCreated":"2018-03-27T11:47:07.520961Z"}]
            03/27/2018 11:47:12> Sending message: 3, Body: [{"machine":{"temperature":21.06818074701269,"pressure":1.0077674268748633},"ambient":{"temperature":20.62468573922509
    7,"humidity":26},"timeCreated":"2018-03-27T11:47:12.5155262Z"}] 
    ```

    Note that each line shows a message with the following format:

    ```json
    {
      "machine": {
        "temperature":21.035142293332676,
        "pressure":1.0040035524049884
      },
      "ambient": {
        "temperature":21.066330931413141,
        "humidity":25
      },
      "timeCreated":"2018-03-27T11:47:02.108829Z"
    }
    ```

1. In another new terminal (keeping the logs running in the other windows), get the `iothubowner` connection string for you hub by running:

    ```bash
    az iot hub show-connection-string --name edge101hub
    ```

    And capture the connection string from the output:

    ```json
    {
      "connectionString": "HostName=edge101hub.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=U6PuHHPPJBl2OD4LcLqSkeoaBytYoH5vYz0molqOWAI="
    }    
    ```

    Get just the connection string:

    ```text
    HostName=edge101hub.azure-devices.net;SharedAccessKeyName=iothubowner;SharedAccessKey=U6PuHHPPJBl2OD4LcLqSkeoaBytYoH5vYz0molqOWAI=
    ```

1. Open the IoT Hub "**Device Explorer**" ([link](https://github.com/Azure/azure-iot-sdk-csharp/tree/master/tools/DeviceExplorer)), and paste the connection string from above and click the "**Update**" button:

    ![Device Explorer Connection String](images/device-explorer-connection-string.png)

1. Switch to the "**Data**" tab, select the `edge101edge` device, then click "**Monitor**".  Soon you should start to see messages arriving:

    ![Device to Cloud Messages](images/device-explorer-device-to-cloud-messages.png)