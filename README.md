# Pipedown
### Router Connectedness Application

##### Pipedown 1.5.1


##### Released November 3, 2016

##### Author: Lisa Roach, Karthik Kumaravel, Quan Le
##### Contact: Please use the Issues page to ask questions or open bugs and feature requests.

## Description

The end goal of the Pipedown is to monitor a CDN router and ensures that it has a stable link to the data center, and if it does not, take it offline by removing its link to the internet.

The application solves the end goal by checking a link that connects back to the data center and ensure that it is healthy. The link is determined to be healthy using [iPerf](https://iperf.fr/) based on parameters such as jitter, bandwidth, packet loss, and dropped packets. If the link is determined to be unhealthy, then the link connecting to the internet would be flushed using [gRPC](http://www.grpc.io/) based on a user defined AS and policy.

#### Prerequisites:

Cisco IOS-XR box running version 6.1.2.12i and above.

Pipedown is meant to be run in a Linux container on the IOS-XR. It has been tested on an Ubuntu 14.04 container running Python 2.7. If you run Pipedown in a different scenario we welcome any feedback you have.

Pipedown attempts to be compatible with Python 3 wherever possible, but since gRPC does not officially support Python 3 yet Python 2.7 should be used. 

#### Current Limitations:

Currently for monitoring multiple links, each link needs to have its own source ip address.

### Vagrant

For an easy Pipedown-in-a-box demonstration, please refer to the [vagrant](https://github.com/cisco-ie/Pipedown/tree/master/vagrant) directory. Here you will be able to download a fully functional vagrant environment that has Pipedown up and running already. This demonstration uses concepts that can be better understood by consulting the IOS XR tutorials: https://xrdocs.github.io/application-hosting/tutorials/.

## Usage

Step 1: Clone this repo and cd into Pipedown

Step 2: It is highly recommended you install and use a [virtualenv](https://virtualenv.pypa.io/en/stable/).

```
pip install virtualenv

virtualenv venv

source venv/bin/activate
```

Step 3: Install gRPC. (If you chose to install outside of a virtualenv, you may have some trouble at this step).

`pip install grpcio`

Step 4 (optional): Install iPerf on both routers. This is only required if you plan to use iPerf as for your
connectivity test. By default, iPerf is in use, so it is recommended that it be installed.

On a linux container, install iPerf how you normally would for your OS. Example:

`apt-get install iperf`

On a native IOS-XR box, use yum:

`sudo yum install iperf`

iPerf will need to be turned on on the data center router whom you are testing connectivity to.

To turn on iPerf on your data center router, use this command:

`iperf -s -B <port address to bind to> -u`


Step 4: Run the `setup.py` script.

`python setup.py install`

Step 5: Configuring the router.

Ensure these things are configured on the router:

- Turn on gRPC.
- Route-policy that will drop everything, (how the app flushes the internet connection).

Step 6: Create a monitor.config file in the router-connectedness directory and fill in the values in the key:value pair.

```
[Name-for-connection]
destination : ip_address           # IP address of where iPerf is running in the data center
source : ip_address                # IP address of your souce link
protocols : protocol               # Protocol you want to monitor [isis, BGP]
bw_thres : bandwidth               # Integer value of Bandwidth in KB that you determine is the minmum value for the link
jitter_thres : jitter_threshold    # Integer value of Jitter Threshold
pkt_loss : packet loss             # Integer value of number of packets allowed to lose
interval : interval                # Integer value in seconds of how often you want the test to run
grpc_server : ip_address           # IP address of the router you are monitoring (Can be local loopback 127.0.0.1)
grpc_port : port                   # gRPC port number
grpc_user : username               # Username for AAA authentication
grpc_pass : password               # Password for AAA authentication
flush_bgp_as : flush_bgp_as        # The AS number of the neighbor group for the internet
drop_policy_name: drop_policy_name # The policy name that you want when the flush is activated.
pass_policy_name: pass_policy_name # The polocy name that you want to pass packets.
text_alert : phone_number          # Phone number to text with text alerting. Turns alerting on.
hostname: hostname                     # Hostname for clarifying alert messages.
```
Example:
```
[IS-IS]
destination : 5.5.5.5
source : 10.1.1.1
protocol : isis
bw_thres : 200
jitter_thres : 20
pkt_loss : 3
interval : 10
grpc_server : 127.0.0.1
grpc_port : 57777
grpc_user : vagrant
grpc_pass : vagrant
flush_as : 65000
drop_policy_name: drop
text_alert : +14087784819
hostname: rtr1
```

Step 7: Turn on iPerf on destination box.

The iPerf server must be running on another router or server (the router to whom you are trying to connect your link) in order to test iPerf.

Use following command to launch iPerf:

```iperf -s -B ip_address -u```

*Replace ip_address with the destination ip address.

Step 8: Run deamon.

Run the monitor daemon. It uses multithreading so a instance will spawn for every link you want to monitor. You can check the log to ensure it is working.

```python monitor_daemony.py```


## Testing

### Unit Tests

Do this at the ~/Pipedown/pipedown directory:


```python -m unittest discover Tests```


**Note**: iPerf must be installed, or two tests will fail. If you do not want to install iPerf just ignore the error messages.


### Integration Tests

**Running iPerf Server**

The iPerf server must be running on another router (the router to whom you are trying to connect your link) in order to test iPerf.

Use following command to launch iPerf:


```iperf -s -B 10.1.1.2 -u```
