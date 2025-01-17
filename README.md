# ot-lan-rest-server


This is a dedicated rest API or RPC-like Linux web app server that can interact with control systems on a local operations technology network (LAN) or intranet inside a building. Supports tls and Basic Auth through the Python Fast API web framework and all BACnet features are handled by the bacpypes3 Python BACnet stack. FastAPI supports Swagger UI out-of-the-box (see tutorials below), which this app fully leverages. This feature can be incredibly useful for poking and prodding an unknown BACnet system within a building. For convenience, IoT edge devices can request data from the app at regular intervals via REST API without the need to delve into any BACnet stack setup.

## Setup Python packages use virtual environment if desired.
```bash
python -m pip install bacpypes3 ifaddr fastapi uvicorn

```

#### Example args to run app on http with setting custom `host` and `port`.
```bash
python app/main.py --host 0.0.0.0 --port 8080
```

#### Example args to run app on http with setting Basic Auth username for the app of `myusername` and password of `mypassword`. Default app username and pass is `admin` and `secret` which should be changed for security purposes.

```bash
python app/main.py --basic-auth-username=myusername --basic-auth-password=mypassword
```

If running your app on **http without the tls arg** log into swagger UI on `http` something like:
* http://xxx.xxx.xxx.xxx:8000/docs

Else if running your app on **https with tls arg** log into swagger UI on `https` something like:
* https://xxx.xxx.xxx.xxx:8000/docs

## Optional TLS support
To ecprypte http web app TCP traffic only, not BACnet which runs on UDP... Generate certs with running the bash script inside the `scripts` directory. Step through the Q/A process for generating the self signed certs about inputing country code, organization, and contact info.

```bash
./scripts/generate_certs.sh
```

#### Example arg to run app with transport layer security (TLS)
* Step through the Q/A to setup your certs with openssl however you want.
```bash
python app/main.py --tls
```


<details>
  <summary>Tutorial for the Fast API Swagger UI</summary>

If you are using credentials on your app with args like `--basic-auth-username=myusername --basic-auth-password=mypassword` on the command line to start the app, proceed to then enter your credentials in green `Authorize` button in the Swagger UI upper right corner else skip this step.

When the app starts successfully dial into the built in Swagger UI feature of Fast API which can be used to test various BACnet commands.

![Alt text](/images/swagger_home.JPG)

</details>

<details>
  <summary>Whois Tutorial via Swagger UI</summary>

Test if the BACnet device responds to a `whois` for the devices BACnet instance ID.
![Alt text](/images/who_is.JPG)

If successful should return:
```bash
[
  {
    "i-am-device-identifier": "device,201201",
    "max-apdu-length-accepted": 286,
    "segmentation-supported": "no-segmentation",
    "vendor-id": 11
  }
]
```

</details>

<details>
  <summary>Read Prop Tutorial via Swagger UI</summary>

Read request to device `201201 analog-value 301 present-value` which is a temperature sensor.

![Alt text](/images/read_prop_pv1.JPG)

If successful should return:
```bash
{
  "success": true,
  "message": "BACnet read request successfully invoked",
  "data": {
    "device_instance": 201201,
    "object_identifier": "analog-input,2",
    "property_identifier": "present-value",
    "read_result": 69.42999267578125
  }
}

```

Read property of a different property_identifier which can be unique to the BACnet device. See bacpypes3 repo basetypes.py for more info:
https://github.com/JoelBender/BACpypes3/blob/main/bacpypes3/basetypes.py

![Alt text](/images/read_prop.JPG)

If successful should return:
```bash
{
  "success": true,
  "message": "BACnet read request successfully invoked",
  "data": {
    "device_instance": 201201,
    "object_identifier": "analog-input,2",
    "property_identifier": "out-of-service",
    "read_result": false
  }
}

```
</details>

<details>
  <summary>Write Prop Tutorial via Swagger UI</summary>

Write request to device `201201 analog-value 301 present-value` for a value of `10` on BACnet priority `10`.
![Alt text](/images/write_req1.JPG)

If successful should return:
```bash
{
  "success": true,
  "message": "BACnet write request successfully invoked",
  "data": {
    "device_instance": 201201,
    "object_identifier": "analog-value,301",
    "property_identifier": "present-value",
    "written_value": 10,
    "priority": 10
  }
}
```

Release an override by passing in the value of `null`. 

![Alt text](/images/write_req2.JPG)

If successful should return:
```bash
{
  "success": true,
  "message": "BACnet write request successfully invoked",
  "data": {
    "device_instance": 201201,
    "object_identifier": "analog-value,301",
    "property_identifier": "present-value",
    "written_value": "null",
    "priority": 10
  }
}
```

</details>

<details>
  <summary>Read Multiple Tutorial via Swagger UI</summary>

A BACnet Read Multiple to device `201201` on a post request would look like this.
![Alt text](/images/rpm.JPG)

With json data in the body with multiple `object_identifier` and `property_identifier`:
```
{
  "device_instance": 201201,
  "requests": [
    {
      "object_identifier": "analog-input,2",
      "property_identifier": "present-value"
    },
    {
      "object_identifier": "analog-input,2",
      "property_identifier": "units"
    },
    {
      "object_identifier": "analog-input,2",
      "property_identifier": "description"
    },
    {
      "object_identifier": "analog-value,301",
      "property_identifier": "present-value"
    },
    {
      "object_identifier": "analog-input,301",
      "property_identifier": "units"
    },
    {
      "object_identifier": "analog-value,301",
      "property_identifier": "description"
    }
  ]
}
```

If successful should return this below. Note that is a property isnt defined inside your BACnet device it will
come back as `error` but if the property does exist it will return a `value`.
```bash
{
  "success": true,
  "message": "BACnet rpm successfully invoked",
  "data": {
    "device_instance": 201201,
    "requests": [
      {
        "object_identifier": "analog-input,2",
        "property_identifier": "present-value",
        "value": "67.7199935913086"
      },
      {
        "object_identifier": "analog-input,2",
        "property_identifier": "units",
        "value": "degrees-fahrenheit"
      },
      {
        "object_identifier": "analog-input,2",
        "property_identifier": "description",
        "error": "property, unknown-property"
      },
      {
        "object_identifier": "analog-value,301",
        "property_identifier": "present-value",
        "value": "nan"
      },
      {
        "object_identifier": "analog-input,301",
        "property_identifier": "units",
        "error": "object, unknown-object"
      },
      {
        "object_identifier": "analog-value,301",
        "property_identifier": "description",
        "error": "property, unknown-property"
      }
    ]
  }
}
```
</details>

<details>
  <summary>BACnet Whois across a range of instance ID's Tutorial via Swagger UI</summary>

A global BACnet `Whois` between range of instance ID's.
![Alt text](/images/who_is_range.JPG)

With json data in the body:
```
{
  "start_instance": 1,
  "end_instance": 300000
}
```


If successful, the response will include known devices, such as the two on my test bench. Please note that this command is intended only for setup purposes of IoT or Building Automation Systems (BAS). It should not be used at short intervals, as it can cause significant disruptions on BACnet systems and potentially cause some devices to go offline. Use this command with caution. If it must be used regularly for security or device health checks, it should not be executed more frequently than once per hour. In BAS contracting, this command is primarily used during the setup phase to gather device configurations for a building automation system.
```bash
[
  {
    "i-am-device-identifier": "device,201201",
    "max-apdu-length-accepted": 286,
    "segmentation-supported": "no-segmentation",
    "vendor-id": 11
  },
  {
    "i-am-device-identifier": "device,201202",
    "max-apdu-length-accepted": 286,
    "segmentation-supported": "no-segmentation",
    "vendor-id": 11
  }
]
```

</details>

<details>
  <summary>Device Point Discovery Tutorial via Swagger UI</summary>

Test for a BACnet point discovery process of a BACnet device by instance ID.
![Alt text](/images/point_discovery.JPG)

If successful, the operation should return all the device objects or points as shown below. Please note that this process may take a while depending on the device, the number of configured points, and the network. Additionally, if the BACnet device supports a BACnet service called object-list, the operation will be faster. If object-list is not supported, the application will read one point at a time, resulting in a longer processing time. However, the response will remain consistent regardless of the method used.

```bash
{
  "success": true,
  "message": "Point discovery successful",
  "data": {
    "device_instance_id": 201201,
    "point_object_details": [
      {
        "identifier": "analog-input 1",
        "description": "tempUoOne10k"
      },
      {
        "identifier": "analog-input 2",
        "description": "tempUoTwoBalco"
      },
      {
        "identifier": "analog-input 3",
        "description": "tempUoThreeBalco"
      },
      {
        "identifier": "analog-input 17",
        "description": "S-LK"
      },
      {
        "identifier": "analog-value 806",
        "description": "RmDiff"
      },
      {
        "identifier": "analog-value 301",
        "description": "Oat"
      },
      {
        "identifier": "analog-value 302",
        "description": "RmTmpSpt"
      },
      {
        "identifier": "analog-value 300",
        "description": "RmTmp"
      },
      {
        "identifier": "binary-output 1",
        "description": "UhCmd"
      },
      {
        "identifier": "binary-value 806",
        "description": "GlblHtgDsbl"
      },
      {
        "identifier": "device 201201",
        "description": "TEST1"
      },
      {
        "identifier": "file 1",
        "description": "Firmware"
      },
      {
        "identifier": "file 32",
        "description": "Application Database"
      },
      {
        "identifier": "multi-state-value 10101",
        "description": "tempUoOne10k:Type"
      },
      {
        "identifier": "analog-value 10106",
        "description": "tempUoOne10k:Filter"
      },
      {
        "identifier": "analog-value 10107",
        "description": "tempUoOne10k:Offset"
      },
      {
        "identifier": "multi-state-value 10201",
        "description": "tempUoTwoBalco:Type"
      },
      {
        "identifier": "analog-value 10206",
        "description": "tempUoTwoBalco:Filter"
      },
      {
        "identifier": "analog-value 10207",
        "description": "tempUoTwoBalco:Offset"
      },
      {
        "identifier": "multi-state-value 10301",
        "description": "tempUoThreeBalco:Type"
      },
      {
        "identifier": "analog-value 10306",
        "description": "tempUoThreeBalco:Filter"
      },
      {
        "identifier": "analog-value 10307",
        "description": "tempUoThreeBalco:Offset"
      },
      {
        "identifier": "binary-value 12501",
        "description": "UhCmd:Action"
      },
      {
        "identifier": "analog-value 13313",
        "description": "S-LK:TempCal"
      },
      {
        "identifier": "analog-value 13332",
        "description": "S-LK:OvrdTm"
      },
      {
        "identifier": "binary-value 13353",
        "description": "S-LK:PbOcc"
      },
      {
        "identifier": "analog-value 13354",
        "description": "S-LK:OvrTimer"
      }
    ]
  }
}
```

</details>


Project goals:
 - [x] who-is request
 - [x] read property request
 - [x] write property request
 - [x] read multiple property request
 - [x] who-is across a range of instance ID's
 - [x] device point discovery
 - [ ] who-is router-to-network
 - [ ] read point proirity array
 - [ ] make a BACnet device toml config file
 - [x] create unit tests for Fast API Pydantic models
 - [x] add pydantic model validation for server requests
 - [ ] read range for BACnet devices that support trend log data
 - [ ] add ModBus support which would be used to read a utility meter only
 
## License
MIT License

Copyright (c) 2024 Ben Bartling

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

ADDITIONAL CYBERSECURITY NOTICE: Users are encouraged to apply the highest level of cybersecurity, OT, IoT, and IT measures when using this software. The authors and copyright holders disclaim any liability for cybersecurity breaches, mechanical equipment damage, financial damage, or loss of life arising from the use of the Software. Users assume full responsibility for ensuring the secure deployment and operation of the Software in their environments.
