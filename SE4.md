SE4 - Simple Standard Service Endpoints
=======================================


Objective
---------

The object of this specification is to provide a standard convention for access to server status, configuration and live health via HTTP (and SPDY if supported by the service).
Services within the Beamly environment must implement this specification in order to be deployed into the production environment.

Important: these endpoints are only intended for internal consumption and not intended to be available externally, doing so may leak sensitive information outside your system.


Resources
---------

The following resources must be implemented:

Name                        | HTTP Verb | URI Path
----------------------------|-----------|-------------------------
[Status](#status)           | GET       | /service/status
[Healthcheck](#healthcheck) | GET       | /service/healthcheck
[GTG (Good to Go)](#gtg)    | GET       | /service/healthcheck/gtg
[Service Canary](#asg)      | GET       | /service/healthcheck/asg


The following resources are desirable:

Name              | HTTP Verb | URI Path
------------------|-----------|----------------
[Config](#config) | GET       | /service/config


<a name="status">Status</a>
---------------------------

The status resource returns information about the service.

Valid Response Codes: 200 OK

Response Media Type: application/json


Element Path     | Required?       | Type      | Description                                                                  | Example
-----------------|-----------------|-----------|------------------------------------------------------------------------------|------------------------------------------
artifact_id      | M               | String    | Artifact name or maven artifact id                                           | "cpt-server"
build_number     | M               | String    | The build pipeline number                                                    | "1537.1"
build_machine    | M               | String    | The machine the artifact was built or verified on                            | "ip-10-4-1-16 (127.0.1.1)"
built_by         | M               | String    | The user that did the build                                                  | "go"
built_when       | M               | String    | When the build was done                                                      | "20140303-1746"
compiler_version | O<sup>1</sup>   | String    | The compiler version                                                         | "1.7.0_51"
current_time     | M               | DateTime  | The current time (time of request)                                           | "2014-03-12T19:40:18.877Z"
git_sha1         | M               | String    | The git sha1 that can be used to identify the primary material for the build | "d567d2650318f704747204815adedd2396a203f5"
group_id         | O               | String    | The maven group id                                                           | "beamly.platform"
machine_name     | M               | String    | The name of the machine responding to this request                           | "ip-10-1-11-196 (127.0.1.1)"
os_arch          | M               | String    | The architecture the OS of the machine responding to the request             | "amd64"
os_avgload       | O               | String    | The average load of the machine responding to the request                    | "0.0"
os_name          | M               | String    | The name of the OS of the machine responding to the request                  | "Linux"
os_numprocessors | O               | String    | The number of processors of the machine responding to the request            | "2"
os_version       | M               | String    | The version of the OS of the machine responding to the request               | "3.2.0-55-virtual"
runbook_uri      | M               | URI       | The URI where the RUNBOOK can be found                                       | "https://XXXXXX/RUNBOOKS/CPT+Runbook"
up_duration      | M               | String    | How long the service responding to the request has been up                   | "730444633 milliseconds"
up_since         | M               | DateTime  | The time at which the service was started                                    | "2014-03-04T08:46:13.877Z"
version          | M               | String    | The version of the service responding to the request	                        | "1537"
vm_name          | O<sup>2</sup>   | String    | The name of the VM that the service is running on                            | "Java HotSpot(TM) 64-Bit Server VM"
vm_vendor        | O<sup>2</sup>   | String    | The vendor of the VM that the service is running on                          | "Oracle Corporation"
vm_version       | O<sup>2</sup>   | String    | The version of the VM that the service is running on                         | "24.51-b03"

 M         Mandatory

 O         Optional

 O<sup>1</sup>   Optional - however mandatory for compiled languages

 O<sup>2 </sup>  Optional - however mandatory for virtual machine based languages


Example:
```
< HTTP/1.1 200 OK
< Server: cpt-server-i/0.0.1/1552 (ip-10-0-1-231 HttpServer2/1552)
< X-Request-Id: req37d516de-cc86-11e3-fedf-ea0b6d27b8ee
< X-Request-Time: 2ms
< Cache-Control: no-cache
< Content-Type: application/json
< Content-Length: 717
<
{
    "artifact_id": "cpt-server",
    "build-number": "1552.1",
    "build_machine": "10-0-10-22",
    "built_by": "go",
    "built_when": "20140417-1342",
    "compiler_version": "1.7.0_51",
    "current_time": "2014-04-25T14:30:58.877Z",
    "git_sha1": "f61f8a375c6a5656a434a011cf93a245815a3e78",
    "group_id": "beamly.platform",
    "machine_name": "ip-10-0-1-231 (127.0.1.1)",
    "os_arch": "amd64",
    "os_avgload": "0.09",
    "os_name": "Linux",
    "os_numprocessors": "1",
    "os_version": "3.2.0-55-virtual",
    "runbook_uri": "https://XXXXXXXXXXXXXXX/XXXXX/RUNBOOKS/CPT+Runbook",
    "up_duration": "283103946 milliseconds",
    "up_since": "2014-04-22T07:52:34.877Z",
    "version": "1552",
    "vm_name": "Java HotSpot(TM) 64-Bit Server VM",
    "vm_vendor": "Oracle Corporation",
    "vm_version": "24.55-b03"
}
```


<a name="healthcheck">Healthcheck</a>
-------------------------------------

The healthcheck resource provides information about internal health and its perceived health of downstream dependencies.

Important: the healthcheck resource must not block waiting for healthcheck probes to execute, it should return the last known status.

Valid response codes: 200 OK

Response Media Type: application/json


Element Path            | Required? | Type          | Description                                                                      | Example
------------------------|-----------|---------------|----------------------------------------------------------------------------------|---------------------------
report_as_of            | M         | DateTime      | The time at which this report was generated (this may not be the current time)   | "2014-03-12T20:16:55.447Z"
report_duration         | M         | String        | How long it took to generate the report                                          | "0 seconds"
tests                   | M         | Array         | array of test results	                                                       |
tests[].duration_millis | M         | Float         | Number of milliseconds taken to run the test                                     | 1.0
tests[].test_name       | M         | String        | The name of the test, a name that is meaningful to supporting engineers          | "CPTCluster"
tests[].test_result     | M         | String (Enum) | The state of the test, may be "not_run", "running", "passed", "failed" | "passed"
tests[].tested_at       | M         | DateTime      | The time at which this test was executed                                         | "2014-03-12T20:16:45.013Z"


Example:

```
< HTTP/1.1 200 OK
< Server: cpt-server-i/0.0.1/1552 (ip-10-0-1-164 HttpServer2/1552)
< X-Request-Id: req941baf96-cc86-11e3-e4b9-add31c37a536
< X-Request-Time: 11ms
< Cache-Control: no-cache
< Content-Type: application/json
< Content-Length: 418
<
{
    "report_as_of": "2014-04-25T14:33:33.383Z",
    "report_duration": "0 seconds",
    "tests": [
        {
            "duration_millis": 1.0,
            "test_name": "Cassandra (CPT)",
            "test_result": "passed",
            "tested_at": "2014-04-25T14:33:15.229Z"
        }
    ]
}
```
 

<a name="gtg">GTG - Good to Go</a>
----------------------------------

The "Good To Go" (GTG) returns a successful response in the case that the service is in an operational state and is able to receive traffic.  This resource is used by load balancers and monitoring tools to determine if traffic should be routed to this service or not.

Note that GTG is not used to determine if the service is healthy or not, only if it is able to receive traffic.  A healthy instance may not be able to accept traffic due to the failure of critical downstream dependencies.

A successful response is a 200 OK with a content of the text "OK" (including quotes) and a media type of "plain/text"

A failed response is a 5XX reponse with either a 500 or 503 response preferred.  Failure to respond within a predetermined timeout typically 2 seconds is also treated as a failure.


```
< HTTP/1.1 200 OK
< Server: cpt-server-i/0.0.1/1552 (ip-10-0-1-164 HttpServer2/1552)
< X-Request-Id: reqd56ff738-cc86-11e3-0144-d52c0ac401f1
< X-Request-Time: 0ms
< Cache-Control: no-cache
< Content-Type: text/plain
< Content-Length: 4
<
"OK"
```


<a name="asg">ASG - Service Canary</a>
--------------------------------------

The "Service Canary" (ASG) returns a successful response in the case that the service is in a healthy state.  If a service returns a failure response or fails to respond within a predefined timeout then the service can expect to be terminated and replaced.  (Typically this resouce is used in auto-scaling group healthchecks.)

A successful response is a 200 OK with a content of the text "OK" (including quotes) and a media type of "plain/text"

A failed response is a 5XX reponse with either a 500 or 503 response preferred.  Failure to respond within a predetermined timeout typically 2 seconds is also treated as a failure.

```
< HTTP/1.1 200 OK
< Server: cpt-server-i/0.0.1/1552 (ip-10-0-11-226 HttpServer2/1552)
< X-Request-Id: reqef5a9cb8-cc86-11e3-c19c-1741c559ab62
< X-Request-Time: 0ms
< Cache-Control: no-cache
< Content-Type: text/plain
< Content-Length: 4
<
"OK"
```


<a name="config">Config</a>
---------------------------

The Config resource returns the configuration that is used by the service.  Typically this is a json representation of the configuration however it is left here as implementation dependent.

Valid response codes: 200 OK

Response Media Type: application/json

