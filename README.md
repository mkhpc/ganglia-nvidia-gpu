## Overview

> Ganglia is an open-source scalable distributed monitoring system for high-performance computing systems such as clusters and Grids. It is carefully engineered to achieve very low per-node overheads and high concurrency. Ganglia is currently in use on thousands of clusters around the world and can scale to handle clusters with several thousand of nodes. For NVIDIA GPUs, there now exists a python module for monitoring NVIDIA GPUs using the newly released Python bindings for NVML (NVIDIA Management Library). These bindings are under BSD license and allow simplified access to GPU metrics like temperature, memory usage, and utilization.

This guide provides the steps to install Ganglia monitoring tool with the configuration of Gmond Python module for GPUs.

## Installation of Ganglia

Ubuntu 18.04/16.04
```console
$ apt-get install -y ganglia-monitor rrdtool gmetad ganglia-webfrontend
$ cp /etc/ganglia-webfrontend/apache.conf /etc/apache2/sites-enabled/ganglia.conf
```

modify gmetad.conf
```console
$ vi /etc/ganglia/gmetad.conf

Change the following

data_source "my cluster" localhost

to

data_source "my cluster" 50 192.168.56.10:8649
```

modify gmond.conf
```console
$ vi /etc/ganglia/gmond.conf

Do the following changes

/* If a cluster attribute is specified, then all gmond hosts are wrapped inside
* of a tag. If you do not specify a cluster tag, then all will
* NOT be wrapped inside of a tag. */
cluster {
name = "unspecified"
owner = "unspecified"
latlong = "unspecified"
url = "unspecified"
}

to

cluster {
name = "my cluster"
owner = "unspecified"
latlong = "unspecified"
url = "unspecified"
}

/* Feel free to specify as many udp_send_channels as you like. Gmond
used to only support having a single channel */
udp_send_channel {
mcast_join = 239.2.11.71
port = 8649
ttl = 1
}

to

/* Feel free to specify as many udp_send_channels as you like. Gmond
used to only support having a single channel */
udp_send_channel {
#mcast_join = 239.2.11.71
host = 192.168.56.10
port = 8649
ttl = 1
}

/* You can specify as many udp_recv_channels as you like as well. */
udp_recv_channel {
mcast_join = 239.2.11.71
port = 8649
bind = 239.2.11.71
}

to

/* You can specify as many udp_recv_channels as you like as well. */
udp_recv_channel {
#mcast_join = 239.2.11.71
port = 8649
#bind = 239.2.11.71
}

save and exit the file
```

Now you need to restart the services
```console
$ /etc/init.d/ganglia-monitor start
$ /etc/init.d/gmetad start
$ /etc/init.d/apache2 restart
```

## Configuration of Gmond Python module for GPUs

modify gmod.conf
```console
$ vi /etc/ganglia/gmond.conf

 59 modules {・
 ・
 ・
 91   module {                                          # add
 92     name = "python_module"                          # add
 93     path = "/usr/lib/ganglia/modpython.so"          # add
 94     params = "/usr/lib/ganglia/python_modules/"     # add
 95   }                                                 # add
 96 }
 97
 98 include ('/etc/ganglia/conf.d/*.conf')
 99 include ('/etc/ganglia/conf.d/*.pyconf')            # add
```

make dir
```console
$ mkdir /etc/ganglia/conf.d
$ mkdir /usr/lib/ganglia/python_modules
```

prepare python module
```consolse
$ git clone https://github.com/mkhpc/ganglia-nvidia-gpu.git
$ cd /ganglia-nvidia-gpu
$ wget https://pypi.python.org/packages/72/31/378ca145e919ca415641a0f17f2669fa98c482a81f1f8fdfb72b1f9dbb37/nvidia-ml-py-7.352.0.tar.gz
$ tar xvfpz nvidia-ml-py-7.352.0.tar.gz
$ cd nvidia-ml-py-7.352.0
$ python setup.py install
$ cd ../  
$ cp -ip python_modules/nvidia.py /usr/lib/ganglia/python_modules/.
$ cp -ip conf.d/nvidia.pyconf /etc/ganglia/conf.d/.
$ cp -ip graph.d/* /usr/share/ganglia-webfrontend/graph.d/.
```

restart services
```console
$ service ganglia-monitor restart 
$ service apache2 restart
```

Now open on browser
```console
http://192.168.56.10/ganglia/
```
