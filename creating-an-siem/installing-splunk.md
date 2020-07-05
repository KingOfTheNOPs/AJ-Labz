---
description: >-
  REF:
  https://github.com/packetiq/SplunkArchitect/blob/master/Install-and-Configure-Splunk-Enterprise-Components.md
---

# packetiq/SplunkArchitect

## Installing Splunk Enterprise on Linux

All Splunk components except a Universal Forwarder \( a separate lightweight package \) are based on an installation of Splunk Enterprise with specific configuration options - so the first step in creating any component in a Splunk solution is installing Splunk Enterprise.

### Obtain the Splunk installation package

* Access the Splunk website:  [Download Splunk Enterprise RPM for Linux-x86\_64](https://www.splunk.com/en_us/download/sem.html?ac=ga_usa_brand_enterprise_exact_Mar17&_kk=splunk%2520enterprise&gclid=CIvWzN6Hk9MCFQsRgQodK_QARg)

**Note: You will have to create an account and/or login using your Splunk user ID & password.**

To fetch the splunk installable directly from a target server, use wget:

* Select the appropriate Splunk download from the website link above
* Cancel the download window
* From the page that appears after the download appears, in the top-right click the link in 'Download via Command Line \(wget\)'
* Copy the wget string and paste it into a command shell

Example wget string:  
 `wget -O splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=6.5.3&product=splunk&filename=splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm&wget=true'`

or...

* Select and download the appropriate rpm package \(...x86\_64.rpm\)
* Use ftp or WinSCP to copy the package to the splunk server.

When you have the installation package on the target server:

* `sudo su` \(root\)
* Change permissions on the file: previous:  
   `-rw-------. 1 baxtj018 admin 226438185 Apr 7 15:13 splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm`

  `chmod 744 splunk-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm`

### Installing the Splunk package

`rpm -i splunk-6.5.1-f74036626f0c-linux-2.6-x86_64.rpm`

You may see:  
 warning: splunk-6.5.3-36937ad027d4-linux-2.6-x86\_64.rpm: Header V4 DSA/SHA1 Signature, key ID 653fb112: NOKEY  
 then:  
 complete

* Start splunk:

  ```text
   sudo su
   cd /opt/splunk/bin  
   ./splunk start  
  ```

* Space-down and answer 'y' to the license question, or use:

`./splunk start --accept-license`

You'll see a series of startup messages, then:

`The Splunk web interface is at http://:8000`

### Logging into Splunk Enterprise - First Time

* Login to Splunk Web: `http://:8000/`

```text
user: admin
password: changeme
(this is the default password upon a new installation)

New password: MyNewPassword
```

You may see a 'Help us improve Splunk software' message...

* un-check the 'Share license usage data with Splunk' box and click 'OK'

Note: You can determine the proper web port with `/opt/splunk/bin/splunk show web-port`

### Post Installation Settings

These are recommended settings to ensure that Splunk starts automatically after a server reboot, set the default environment variable, and

**Note: all splunk commands are run from /opt/splunk/bin**

* Configure linux to start Splunk on boot:

`./splunk enable boot-start`

You'll get messages:  
 Init script installed at /etc/init.d/splunk. Init script is configured to run at boot.

The enable boot-start exercise above creates a 'splunk' start-up script file in /etc/rc.d/init.d

#### Disable Transparent Huge Pages

The /etc/rc.d/init.d file should be edited to add functions to disable Transparent Huge Pages on some linux distros such as Red Hat.

[Instructions](https://github.com/packetiq/SplunkArchitect/blob/master/DisableTransparentHugePages.md)

#### Set the environment variable for $SPLUNK\_HOME

Append this to the bottom of /etc/profile file:

`export SPLUNK_HOME="/opt/splunk"`

and run this command from the command line as well to create the variable in the current session.

You can now do

`$SPLUNK_HOME/bin/` or `cd $SPLUNK_HOME`

#### Change the default Splunk Web port from 8000 to 8080 \(Optional\)

For an initial test scenario, you may want to use non-SSL access, on a non-standard port. **See the next section to configure a different port and/or use SSL**

`/opt/splunk/bin/splunk set web-port 8080`

This adds the following to the **web.conf** file in `opt/splunk/etc/system/local/`:

```text
[settings]
httpport = 8080  
```

### Configure Splunk Web to use SSL \(Recommended\)

From Splunk Web:

Settings &gt; Server settings &gt; General settings Splunk Web

Run Splunk Web: Yes \(on by default\) Enable SSL \(HTTPS\) in Splunk Web?: Yes Web port: 8443 Click Save This will modify the **web.conf** file in `/opt/splunk/etc/system/local/` to contain:

```text
[settings]
httpport = 8443
enableSplunkWebSSL = true
```

#### Synchronize system clocks across the distributed search environment

Synchronize the system clocks on all machines, virtual or physical, that are running Splunk Enterprise distributed search instances. Specifically, this means your search heads and search peers. In the case of search head pooling or mounted bundles, this also includes the shared storage hardware. Otherwise, various issues can arise, such as bundle replication failures, search failures, or premature expiration of search artifacts.

The synchronization method that you use depends on your specific set of machines. For most environments, Network Time Protocol \(NTP\) is the best approach.

### Forwarding internal data to search peers

You will want to forward internal log data \(\_internal, \_audit, etc.\) of any Splunk component in a distributed environment \(except Universal Forwarders\) to the indexer cluster so that this data can be searched, as follows:

* Create or edit an outputs.conf file in `/opt/splunk/etc/system/local/` that configures the search head for load-balanced forwarding across the set of search peers \(indexers\).
* Turn off indexing on the Splunk component so that it does not both retain the data locally as well as forward it to the search peers.

Here is an example outputs.conf file:

```text
# Turn off indexing (except on indexers)
[indexAndForward]
index = false
 
[tcpout]
defaultGroup = my_search_peers 
forwardedindex.filter.disable = true  
indexAndForward = false 
 
[tcpout:my_search_peers]
server=10.10.10.1:9997,10.10.10.2:9997,10.10.10.3:9997
autoLB = true
```

**You are now ready to customize this instance of Splunk Enterprise for a specific function**

The following sections outline how to configure instances of Splunk Enterprise to perform specific functions \(search head, indexer, etc.\) in a distributed and/or clustered Splunk solution.

It may be helpful to read the section '[Overview of a Clustered Splunk Environment]()' below before progressing to specific component configuration.

If you are using a standalone instance of Splunk Enterprise you may want to apply some of the function-specific settings outlined in the following sections. Generic settings that may be applied to any Splunk solution will be indicated.

### Cleaning an Index

\_\_!!! DANGEROUS - CANNOT BE UNDONE!!! \_\_

`splunk stop`

To permanently remove event data from all indexes \(**why?**\), type: `splunk clean eventdata` To permanently remove event data from a single index, type: `splunk clean eventdata -index yourindex`

`splunk start`

### Splunk README Files

The `/opt/splunk/etc/system/README` directory of any Splunk installation contains a complete set of text documents for all of the Splunk .conf files. There are two files for each configuration:

```text
.conf.example - annotated examples
.conf.spec - detailed description of options
```

A copy of the README files for Splunk 6.5 are in the README directory of this repository.

The more useful of these may include:

[distsearch.conf.example](https://github.com/packetiq/SplunkArchitect/blob/master/README/distsearch.conf.example)  
 [distsearch.conf.spec](https://github.com/packetiq/SplunkArchitect/blob/master/README/distsearch.conf.spec)  
 [indexes.conf.example](https://github.com/packetiq/SplunkArchitect/blob/master/README/indexes.conf.example)  
 [indexes.conf.spec](https://github.com/packetiq/SplunkArchitect/blob/master/README/indexes.conf.spec)  
 [inputs.conf.example](https://github.com/packetiq/SplunkArchitect/blob/master/README/inputs.conf.example)  
 [inputs.conf.spec](https://github.com/packetiq/SplunkArchitect/blob/master/README/inputs.conf.spec)  
 [outputs.conf.example](https://github.com/packetiq/SplunkArchitect/blob/master/README/outputs.conf.example)  
 [outputs.conf.spec](https://github.com/packetiq/SplunkArchitect/blob/master/README/outputs.conf.spec)  
 [props.conf.example](https://github.com/packetiq/SplunkArchitect/blob/master/README/props.conf.example)  
 [props.conf.spec](https://github.com/packetiq/SplunkArchitect/blob/master/README/props.conf.spec)  
 [serverclass.conf.example](https://github.com/packetiq/SplunkArchitect/blob/master/README/serverclass.conf.example)  
 [serverclass.conf.spec](https://github.com/packetiq/SplunkArchitect/blob/master/README/serverclass.conf.spec)  
 [transforms.conf.example](https://github.com/packetiq/SplunkArchitect/blob/master/README/transforms.conf.example)  
 [transforms.conf.spec](https://github.com/packetiq/SplunkArchitect/blob/master/README/transforms.conf.spec)

## Uninstalling Splunk Enterprise on Linux

This process is the same for uninstalling Splunk Enterprise or a Universal Forwarder:

Get package name: `rpm -q -a | grep -i splunk`

Uninstall: `rpm -e` Example: `rpm -e splunk-6.5.3-36937ad027d4.x86_64`

Clean up remaining files/directory: `cd /opt` `rm -rf ./splunk`

[top]()

## Overview of a Clustered Splunk Solution

**Indexer clustering**

Splunk supports the concept of utilizing multiple indexers in an 'index cluster' to increase indexing capability and to allow setting a 'replication factor' which creates multiple, redundant copies of rawdata files across multiple indexers so that if an indexer fails data is not lost, as well as a 'search factor' wherein multiple copies of data indexes are distributed as well so that search performance is not affected if an indexer fails.

The image below depicts a simple clustered indexer solution:

There are two types of nodes in an indexer cluster:

* A single Cluster Master to manage the cluster. This 'master node' is a specialized type of indexer that, while it doesn't participate in any data streaming, coordinates a range of activities involving the search peers and the search head, including coordinating which 'buckets' are replicated across the peer nodes for redundancy.
* Several 'peer nodes' \(indexers\) that handle the indexing function for the cluster, indexing and maintaining multiple copies of the rawdata and index files, and running searches across the data.

#### Index Time vs Search Time Processing

Splunk Enterprise documentation contains references to the terms "index time" and "search time". These terms distinguish between the types of processing that occur during indexing, and the types that occur when a search is run.

It is better to perform most knowledge-building activities, such as field extraction, at search time. Index-time custom field extraction can degrade performance at both index time and search time.

Index-time processes take place between the point when the data is consumed and the point when it is written to disk.

The following processes occur during index time \(controlled by **props.conf** on an indexer\):

* Default field extraction \(such as host, source, sourcetype, and timestamp\)
* Static or dynamic host assignment for specific inputs
* Default host assignment overrides
* Source type customization
* Custom index-time field extraction
* Structured data field extraction
* Event timestamping
* Event linebreaking
* Event segmentation \(also happens at search time\)

Search-time processes take place while a search is run, as events are collected by the search. The following processes occur at search time:

* Event segmentation \(also happens at index time\)
* Event type matching
* Search-time field extraction \(automatic and custom field extractions, including multivalue fields and calculated fields\)
* Field aliasing
* Addition of fields from lookups
* Source type renaming
* Tagging

[top]()

## Create a Splunk Indexer

Splunk Enterprise stores all of the data it processes in indexes. An index is a collection of databases, which are subdirectories located in `$SPLUNK_HOME/var/lib/splunk`.

Indexes consist of two types of files: compressed **rawdata files** and **index files**.

Splunk Enterprise comes with a number of preconfigured indexes, including:

* main: This is the default Splunk Enterprise index. All processed data is stored here unless otherwise specified.
* \_internal: Stores Splunk Enterprise internal logs and processing metrics.
* \_audit: Contains events related to the file system change monitor, auditing, and all user search history.

**To enable an indexer as a peer node:**

1. Click Settings in the upper right corner of Splunk Web.
2. In the Distributed environment group, click Indexer clustering.
3. Select Enable indexer clustering.
4. Select Peer node and click Next.
5. There are a few fields to fill out:

* Master URI. Enter the master's URI, including its management port. For example: [https://10.152.31.202:8089](https://10.152.31.202:8089/).
* Peer replication port. This is the port on which the peer receives replicated data streamed from the other peers. You can specify any available, unused port for this purpose. This port must be different from the management or receiving ports.
* Security key. This is the key that authenticates communication between the master and the peers and search heads. The key must be the same across all cluster nodes. Set the same value here that you previously set on the master node.

1. Click Enable peer node.

The message appears, "You must restart Splunk for the peer node to become active."

1. Click Go to Server Controls. This takes you to the Settings page where you can initiate the restart.
2. Repeat this process for all the cluster's peer nodes.

When you have enabled the 'replication factor' number of peers, the cluster can start indexing and replicating data \(it will be blocked by the master node until the RF number of peers is enabled\).

**Configure peer nodes with server.conf**

You can also configure an indexer / peer node my editing the server.conf file in `/opt/splunk/etc/system/local/`; note the 'mode = slave' entry:

```text
[replication_port://9887]

[clustering]
master_uri = https://10.152.31.202:8089
mode = slave
pass4SymmKey = whatever
```

This example specifies that:

* the peer will use port 9887 to listen for replicated data streamed from the other peers. You can specify any available, unused port as the replication port. Do not re-use the management or receiving ports.
* the peer's cluster master resides at 10.152.31.202:8089.
* the instance is a cluster peer \("slave"\) node.
* the security key is "whatever".

### Configure inputs on the peers

If you decide not to use forwarders to handle your data inputs, you can set up inputs on each peer in the usual way; for example, by editing inputs.conf on the peers.

In Splunk Web: Settings &gt; Forwarding and receiving &gt; Configure Receiving &gt; New Listen on this port: 9997

When you add an input through Splunk Web, Splunk Enterprise adds that input to a copy of inputs.conf. The app context, that is, the Splunk app you are currently in when you configure the input, determines where Splunk Enterprise writes the inputs.conf file.

For example, if you navigated to the Settings page directly from the Search page and then added an input, Splunk Enterprise adds the input to `op/splunk/etc/apps/search/local/inputs.conf`:

```text
[splunktcp://9997]
connection_host = ip
```

[top]()

## Create a Cluster Master

If you are going to create a indexer cluster, you must also create a 'master node' \(Splunk nomenclature\) - aka a Cluster Master - to manage the cluster.

**To configure a Splunk Enterprise instance as the Cluster Master / master node:**

1. Click Settings in the upper right corner of Splunk Web.
2. In the Distributed environment group, click Indexer clustering.
3. Select Enable indexer clustering.
4. Select Master node and click Next.
5. There are a few fields to fill out:

* Replication Factor. The replication factor determines how many copies of data the cluster maintains. The default is 3. Be sure to choose the right replication factor now. It is inadvisable to increase the replication factor later, after the cluster contains significant amounts of data.
* Search Factor. The search factor determines how many immediately searchable copies of data the cluster maintains. The default is 2. Be sure to choose the right search factor now. It is inadvisable to increase the search factor later, once the cluster has significant amounts of data.
* Security Key. This is the key that authenticates communication between the master and the peers and search heads. The key must be the same across all cluster nodes. The value that you set here must be the same that you subsequently set on the peers and search heads as well.
* Cluster Label. You can label the cluster here. The label is useful for identifying the cluster in the monitoring console. See Set cluster labels in Monitoring Splunk Enterprise.

1. Click Enable master node. The message appears, "You must restart Splunk for the master node to become active. You can restart Splunk from Server Controls."
2. Create an outputs.conf file in `/opt/splunk/etc/system/local/` on the master node that configures it for load-balanced forwarding of its internal log files across the set of peer nodes. You must also turn off indexing on the master, so that the master does not both retain the data locally as well as forward it to the peers.

   [Forward internal data to search peers]()

3. Click Go to Server Controls. This takes you to the Settings page where you can initiate the restart.

   Important: When the master starts up for the first time, it will block indexing on the peers until you enable and restart the full replication factor number of peers. Do not restart the master while it is waiting for the peers to join the cluster. If you do, you will need to restart the peers a second time.

4. After the restart, log back into the master and return to the Clustering page in Splunk Web \(Settings &gt; Indexer Clustering&gt;\). You will see the master clustering dashboard.

**Configuring a Cluster Master using server.conf**

Enabling a Cluster Master or editing its configuration can also be done in `/opt/splunk/etc/system/local/server.conf`:

```text
[clustering]
mode = master
replication_factor = 4
search_factor = 3
pass4SymmKey = whatever
cluster_label = cluster1
```

### Multisite indexer clusters

Multisite indexer clusters allow you to maintain complete copies of your indexed data in multiple locations. This offers the advantages of enhanced disaster recovery and search affinity. You can specify the number of copies of data on each site. Multisite clusters are similar in most respects to basic, single-site clusters, with some differences in configuration and behavior.

For multisite clusters, you must also take into account the search head and peer node requirements of each site, as determined by your search affinity and disaster recovery needs. At a minimum, you will need \(replication factor + 2\) instances.

**Configure the master node**

You configure the key attributes for the entire cluster on the master node in `/opt/splunk/etc/system/local/server.conf`. Here is an example of a multisite configuration for a master node:

```text
[general]
site = site1

[clustering]
mode = master
multisite = true
available_sites = site1,site2
site_replication_factor = origin:2,total:3
site_search_factor = origin:1,total:2
pass4SymmKey = whatever
cluster_label = cluster1
```

This example specifies that:

* the instance is located on site1.
* the instance is a cluster master node.
* the cluster is multisite.
* the cluster consists of two sites: site1 and site2.
* the cluster's replication factor is the default "origin:2,total:3".
* the cluster's search factor is "origin:1,total:2".
* the cluster's security key is "whatever".
* the cluster label is "cluster1."

Note the following:

* You specify the site attribute in the \[general\] stanza.
* You specify all other multisite attributes in the \[clustering\] stanza.
* You can locate the master on any site in the cluster, but each cluster has only one master.
* You must set multisite=true.
* You must list all cluster sites in the available\_sites attribute.
* You must set a site\_replication\_factor and a site\_search\_factor. For details, see Configure the site replication factor and Configure the site search factor.
* The pass4SymmKey attribute, which sets the security key, must be the same across all cluster nodes. See Configure the indexer cluster with server.conf for details.
* The cluster label is optional. It is useful for identifying the cluster in the monitoring console.

### Managing Configurations Across Peers

The peer update process ensures that all peer nodes share a common set of key configuration files. You must manually invoke this process to distribute and update common files, including apps, to the peer nodes. The process also runs automatically when a peer joins the cluster.

The configuration files that must be identical in most circumstances are indexes.conf, props.conf, and transforms.conf. Other configuration files can also be identical, depending on the needs of your system. Since apps usually include versions of those key files, you should also maintain a common set of apps across all peers.

The set of configuration files and apps common to all peers is called the configuration bundle. The process used to distribute the configuration bundle is known as the configuration bundle method.

To distribute new or edited configuration files or apps across all the peers, you add the files to the configuration bundle on the master and tell the master to distribute the files to the peers using the command: `splunk apply cluster-bundle`.

On the master, the configuration bundle resides under the $SPLUNK\_HOME/etc/master-apps directory. The set of files under that directory constitute the configuration bundle. They are always distributed as a group to all the peers. The directory has this structure:

```text
$SPLUNK_HOME/etc/master-apps/
     _cluster/
          default/
          local/
     /
     /
     ...
```

The /\_cluster directory is a special location for configuration files that need to be distributed across all peers, as follows:

* The /\_cluster/default subdirectory contains a default version of indexes.conf. Do not add any files to this directory and do not change any files in it. This peer-specific default indexes.conf has a higher precedence than the standard default indexes.conf, located under `$SPLUNK_HOME/etc/system/default`.
* The `/_cluster/local` subdirectory is where you can put new or edited configuration files that you want to distribute to the peers.
* When you then restart the master following completion of the upgrade, it performs a rolling restart on its peer nodes and pushes the new bundle to the `/slave-apps` directory on all the peer nodes.
* The `/` subdirectories are optional. They provide a way to distribute any app to the peer nodes. Create and populate them as needed. For example, to distribute "appBestEver" to the peer nodes, place a copy of that app in its own subdirectory: `$SPLUNK_HOME/etc/master-apps/appBestEver`.
* To delete an app that you previously distributed to the peers, remove its directory from the configuration bundle. When you next push the bundle, the app will be deleted from each peer.
* Note: The master only pushes the contents of subdirectories under master-apps. It will not push any standalone files directly under master-apps.
* You explicitly tell the master when you want it to distribute the latest configuration bundle to the peers. In addition, when a peer registers with the master \(for example, when the peer joins the cluster\), the master distributes the current configuration bundle to it.

Caution: When the master distributes the bundle to the peers, it distributes the entire bundle, overwriting the entire contents of any configuration bundle previously distributed to the peers.

**The master-apps location is only for peer node files**. The master does not use the files in that directory for its own configuration needs.

### Configuring indexes.conf on a Cluster Master

If you are using a index cluster, you must configure custom indexes in an indexes.conf file on the Cluster Master in a `/opt/splunk/etc/master-apps//local/` directory.

The `maxTotalDataSizeMB` and `frozenTimePeriodInSecs` attributes in indexes.conf determine when buckets roll from cold to frozen.

If you set the coldToFrozenDir attribute in indexes.conf, the indexer will automatically copy frozen buckets to the specified location before erasing the data from the index.

The following is an example of typical custom index entries in an indexes.conf file:

```text
[]
homePath   = volume:primary/index_name/db
coldPath   = volume:primary/index_name/colddb
thawedPath = $SPLUNK_DB/index_name/thaweddb
# Seting the repFactor attribute to "auto" causes the index's data to be replicated to other peers in the cluster
# By default, repFactor is set to 0, which means that the index will not be replicated
repFactor = auto
# index size of 30 GB = 30 x 1024 MB
maxTotalDataSizeMB = 30720
# only keep data for 30 days before archiving (default is 6 years)
frozenTimePeriodInSecs = 2592000
# you can specify a frozen archive location
# if no fozen archive location is specified, the data is discarded
# coldToFrozenDir = ""
```

### Configuring props.conf files

The props.conf \(and transforms.conf, if applicable\) file is distributed across the indexers \('search peers'\) to tell Splunk how to parse incoming data. You configure props.conf & transforms.conf in the `/opt/splunk/etc/master-apps//local/` directories.

When the cluster bundle is applied \(distributed to the indexers / search peers\) these same files will be located in `/opt/splunk/etc/apps//local/` directories on each indexer.

Here is an assorted example of some props.conf entries; check the options for each of these entries in the [props.conf.example](https://github.com/packetiq/SplunkArchitect/blob/master/README/props.conf.example) or [props.conf.spec](https://github.com/packetiq/SplunkArchitect/blob/master/README/props.conf.spec) files:

```text
BREAK_ONLY_BEFORE = ([\r\n]+)
DATETIME_CONFIG = 
MAX_TIMESTAMP_LOOKAHEAD = 30
NO_BINARY_CHECK = true
TIME_FORMAT = %Y-%m-%dT%H:%M:%S.%3N
TIME_PREFIX = timestamp="
TRUNCATE = 0
SHOULD_LINEMERGE = false
LINE_BREAKER = ([\r\n]+)\[\d{4}-\w{3}-\d{1,2}\s\d{1,2}:\d{1,2}:\d{1,2}\.\d{1,3}\]\s\w+
PREAMBLE_REGEX = ^log4j\:[^\r\n]+[\r\n][^:]+[^\s]+[^\r\n]
KV_MODE = json

# Used with a transforms.conf file
# ID the sourcetype to perform the transform on:
[sourcetype::mssqlserver]
# Then TRANSFORMS- = 
TRANSFORMS-DBHost = db_host
```

### Configuring transforms.conf files

The transforms.conf file is used to

Here are some assorted samples of transforms.conf entires; check the options for each of these entries in the [transforms.conf.example](https://github.com/packetiq/SplunkArchitect/blob/master/README/transforms.conf.example) or [transforms.conf.spec](https://github.com/packetiq/SplunkArchitect/blob/master/README/transforms.conf.spec) files:

```text
# Used with the props.conf file in the previous example
# [db_host] matches with `TRANSFORMS-DBHost = db_host' in props.conf
[db_host]
REGEX = MachineName="(?.*?)"
# Note that 'hostname' must match the RegEx field name above
FORMAT = hostname::$1
# use MetaData:Host to replace the default host entry in an event
DEST_KEY = MetaData:Host


# Another example
[get_path]
SOURCE_KEY = uri
REGEX = ^(?[^\s\?]+) 

[get_query_string]
SOURCE_KEY = uri
REGEX = \?([^\s]+) 
```

### Applying a cluster bundle

\_\_To distribute a new configuration bundle: \_\_

* Validate the bundle: `splunk validate cluster-bundle`
* You can check the status of bundle validation `splunk show cluster-bundle-status`
* Apply the bundle: `splunk apply cluster-bundle`
* To avoid having to answer the prompt: `splunk apply cluster-bundle --answer-yes`

**NOTE: Be sure to validate the cluster bunde before running `splunk apply` - if you apply a bundle with errrors in the indexes.conf file you could bring the entire indexer cluster down, necessitating fixing the error manually on each indexer and restarting it.**

You can also distribute a new configuration bundle from the GUI on the Cluster Master:

* Settings &gt; Indexer clustering
* Click Edit &gt; Distribute Configuration Bundle A dashboard appears with information on the last successful push.
* Click the Distribute Configuration Bundle button. A pop-up window warns you that the distribution might, under certain circumstances, initiate a restart of all the peer nodes. For information on which configuration changes cause a peer restart, see Restart or reload after configuration bundle changes?.
* Push Changes

Some changes to files in the configuration bundle require that the peers restart. In other cases, the peers can just reload, avoiding any interruption to indexing or searching. The bundle reload phase on the peers determines whether a restart is required and directs the master to initiate a rolling restart of the peers only if necessary.

A Reload occurs when:

* If you are using load-balanced forwarders to send data to the peer nodes and you make any changes to props.conf or transforms.conf.
* If you are not using load-balanced forwarders and
* you add a new sourcetype in props.conf.
* you add or update any TRANSFORMS- stanzas in props.conf.
* you add any new stanzas in transforms.conf.
* You make any of these changes in indexes.conf:
  * Adding new index stanzas
  * Enabling or disabling an index with no data
  * Changing any attributes not listed as requiring restart in Determine which indexes.conf changes require restart.

A Restart occurs when:

* The configuration bundle contains changes to any configuration files besides indexes.conf, props.conf, or transforms.conf.
* You make any changes to props.conf or transforms.conf, other than those specified in the reload list, above.
* You make any of the indexes.conf changes described in Determine which indexes.conf changes require restart.
* You delete an existing app from the configuration bundle.

### Restarting a Cluster Master or peers

When restarting cluster peers, you should use Splunk Web or one of the cluster-aware CLI commands, such as splunk offline or splunk rolling-restart. Do not use splunk restart.

If, for any reason, you do need to restart both the master and the peer nodes:

* Restart the master node using **'splunk restart'**
* Restart the peers as a group, using **'splunk rolling-restart cluster-peers'**

You might occasionally have need to restart a single peer; for example, if you change certain configurations on only that peer. There are two ways that you can safely restart a single peer:

* Use Splunk Web \(Settings&gt;Server Controls\).
* Run the command **'splunk offline'**, followed by **'splunk start'**.

When you use Splunk Web or the splunk offline/splunk start commands to restart a peer, the master waits 60 seconds \(by default\) before assuming that the peer has gone down for good. This allows sufficient time for the peer to come back on-line and prevents the cluster from performing unnecessary remedial activities.

**Cluster Master failure**

If a master node goes down, peer nodes can continue to index and replicate data, and the search head can continue to search across the data, for some period of time. Problems eventually will arise, however, particularly if one of the peers goes down. There is no way to recover from peer loss without the master, and the search head will then be searching across an incomplete set of data. Generally speaking, the cluster continues as best it can without the master, but the system is in an inconsistent state and results cannot be guaranteed.

[top]()

## Create a Splunk Search Head

There are two basic options for deploying a distributed search environment:

* One or more independent search heads to search across the search peers.
* Multiple search heads in a search head cluster.

In either case, you must set the initial configuration to create a search head:

**Configure a single Splunk instance as a search head in an indexer cluster:**

1. Click Settings in the upper right corner of Splunk Web.
2. In the Distributed environment group, click Indexer clustering.
3. Select Enable clustering. You will get selections:
   * Master node
   * Peer node
   * Search head node
4. Select Search head node and click Next.
5. There are a few fields to fill out:

* Master URI. Enter the master's URI, including its management port. For example: [https://10.152.31.202:8089](https://10.152.31.202:8089/).
* Security key. This is the key that authenticates communication between the master and the peers and search heads. The key must be the same across all cluster nodes. Set the same value here that you previously set on the master node. An example of a security key is `MyS3cur1tyk3y`

1. Click Enable search head node.

   The message appears, "You must restart Splunk for the search node to become active. You can restart Splunk from Server Controls."

2. Click Go to Server Controls. This takes you to the Settings page where you can initiate the restart.
3. After the restart, log back into the search head and return to the Clustering page in Splunk Web. This time, you see the search head's clustering dashboard.
4. Create an outputs.conf file on the search head that configures the search head for load-balanced forwarding across the set of search peers \(indexers\). You must also turn off indexing on the search head, so that the search head does not both retain the data locally as well as forward it to the search peers.

   [Forward internal data to search peers]()

**Creating a search head by editing server.conf:**

You can also create a search head by editing the **server.conf** file in `/opt/splunk/etc/system/local`; note the 'mode = searchhead' entry:

```text
[clustering]
master_uri = https://10.152.31.202:8089
mode = searchhead
pass4SymmKey = whatever
```

This example specifies that:

* the search head's cluster master resides at 10.152.31.202:8089.
* the instance is a cluster search head.
* the security key is "whatever".

### Distributed search with single dedicated search head

This is a configuration with a single dedicated search head connected to an indexer cluster.

1. Designate a Splunk Enterprise instance as the search head. Since distributed search is enabled automatically on every full Splunk Enterprise instance, you do not actually perform any action in this step, aside from choosing the instance that you want to be your search head.
2. Establish connections from the search head to all the search peers that you want it to search across. This is the key step in the procedure:

   To activate distributed search, you add search peers, or indexers, to a Splunk Enterprise instance that you designate as a search head. You do this by specifying each search peer manually:

* Settings &gt; Distributed search &gt; Search peers &gt; New
* Specify the search peer, along with any authentication settings Note: You must precede the search peer's host name or IP address with the URI scheme, either "http" or "https".
* Save
* Repeat for each of the search head's search peers.

1. Log in to the search head and perform a search that runs across all the search peers, such as a search for \*. Examine the 'splunk\_server' field in the results to verify that all the search peers are listed in that field.

To increase the search management capacity, deploy multiple search heads as members of a search head cluster.

### Creating a Search Head Cluster

A search head cluster is a group of search heads that work together to provide scalability and high availability. The search heads in the cluster share resources, configurations, and jobs. This offers a way to scale a deployment transparently to users.

The search heads in a cluster are, for most purposes, interchangeable. All search heads have access to the same set of search peers. They can also run or access the same searches, dashboards, knowledge objects, and so on.

In each case, the search heads perform only the search management and presentation functions. They connect to search peers \(indexers\) that index data and search across the indexed data.

This configuration is depicted in the following image:

When initiating a distributed search, the search head replicates and distributes its knowledge objects to its search peers \(indexers\) so that they can properly execute queries on its behalf. Knowledge objects include saved searches, event types, and other entities used in searching across indexes; this set of knowledge objects is called the knowledge bundle.

Bundles typically contain a subset of files \(configuration files and assets\) from `$SPLUNK_HOME/etc/system`, `$SPLUNK_HOME/etc/apps`, and `$SPLUNK_HOME/etc/users`.

The process of distributing knowledge bundles means that peers by default receive nearly the entire contents of the search head's apps. If an app contains large binaries that do not need to be shared with the peers, you can eliminate them from the bundle and thus reduce the bundle size.

On the search head, the knowledge bundles resides under the `$SPLUNK_HOME/var/run` directory. The bundles have the extension .bundle for full bundles or .delta for delta bundles. They are tar files, so you can run tar tvf against them to see the contents.

The knowledge bundle gets distributed to the `$SPLUNK_HOME/var/run/searchpeers` directory on each search peer.

On a search head cluster, you can view replication status from the search head cluster captain:  
 Settings &gt; Distributed Search &gt; Search Peers

**1. Create a Deployer first**

To update member configurations of a search head cluster, you need a Splunk Enterprise instance that functions as the deployer. See: [Create a Deployer]()

#### Configure the Search Head cluster members

For each instance that you want to include in the search head cluster, run the `splunk init shcluster-config` command and restart the instance:

```text
splunk init shcluster-config -auth : -mgmt_uri : -replication_port  -replication_factor  -conf_deploy_fetch_url : -secret  -shcluster_label 
```

For example:

```text
splunk init shcluster-config -auth admin:changed -mgmt_uri https://sh1.example.com:8089 -replication_port 34567 -replication_factor 2 -conf_deploy_fetch_url https://10.160.31.200:8089 -secret mykey -shcluster_label shcluster1

splunk restart 
```

\_\_Notes: \_\_

* This command is only for cluster members. Do not run this command on the deployer.
* The -auth parameter specifies your current login credentials for this instance. This parameter is required.
* The -mgmt\_uri parameter specifies the URI and management port for this instance. You must use the fully qualified domain name. This parameter is required.
* The -replication\_port parameter specifies the port that the instance uses to listen for search artifacts streamed from the other cluster members. You can specify any available, unused port as the replication port. Do not reuse the instance's management or receiving ports. This parameter is required.
* The -replication\_factor parameter determines the number of copies of each search artifact that the cluster maintains. All cluster members must use the same replication factor. This parameter is optional. If not explicitly set, the replication factor defaults to 3.
* The -conf\_deploy\_fetch\_url parameter specifies the URL and management port for the deployer instance. This parameter is optional during initialization, but you do need to set it before you can use the deployer functionality. See "Use the deployer to distribute apps and configuration updates."
* The -secret parameter specifies the security key that authenticates communication between the cluster members and between each member and the deployer. The key must be the same across all cluster members and the deployer.

**Required number of instances**

The cluster must contain at a minimum the number of members needed to fulfill both of these requirements:

* Three members, so that the cluster can continue to function if one member goes down.
* The replication factor number of instances. See Choose the replication factor for the search head cluster. For example, if your replication factor is either 2 or 3, you need at least three instances. If your replication factor is 5, you need at least five instances.

You can optionally add more members to boost search and user capacity. Search head clustering supports up to 50 members in a single cluster.

**Ports that the cluster members use**

These ports must be available on each member:

* The management port \(by default, 8089\) must be available to all other members.
* The http port \(by default, 8000\) must be available to any browsers accessing data from the member.
* The KV store port \(by default, 8191\) must be available to all other members. You can use the CLI command `splunk show kvstore-port` to identify the port number.
* The replication port must be available to all other members.

These ports must be in your firewall's list of allowed ports.

Caution: Do not change the management port on any of the members while they are participating in the cluster. If you need to change the management port, you must first remove the member from the cluster.

Select one of the initialized instances to be the first cluster captain. It does not matter which instance you select for this role.

**2. Run the splunk bootstrap shcluster-captain command on a selected instance:**

`splunk bootstrap shcluster-captain -servers_list ":,:,..." -auth:`

Note the following:

* This command designates the specified instance as the first cluster captain. Run this command on only a single instance.
* The -servers\_list parameter contains a comma-separated list of the cluster members, including the member that you are running the command on. The members are identified by URI and management port. This parameter is required.
* Important: The URIs that you specify in -servers\_list must be exactly the same as the ones that you specified earlier when you initialized each member, in the -mgmt\_uri parameter. You cannot, for example, use `https://foo.example.com:8089` during initialization and `https://foo.subdomain.example.com:8089` here, even if they resolve to the same node.

Here is an example of the bootstrap command:

```text
splunk bootstrap shcluster-captain -servers_list "https://sh1.example.com:8089,https://sh2.example.com:8089,https://sh3.example.com:8089,https://sh4.exam
```

**Search Head Pools** You cannot enable search head clustering on an instance that is part of a search head pool. For information on migrating, see [Migrate from a search head pool to a search head cluster](http://docs.splunk.com/Documentation/Splunk/6.5.3/DistSearch/Migratefromsearchheadpooling)

**distsearch.conf**

The settings available through Splunk Web provide sufficient options for most configurations. Some advanced configuration settings, however, are only available by directly editing distsearch.conf. This section discusses only the configuration settings necessary for connecting search heads to search peers. For information on the advanced configuration options, see the distsearch.conf spec file.

**Add the search peers**

1. On the search head, create or edit a distsearch.conf file in `$SPLUNK_HOME/etc/system/local`.
2. Add the search peers to the servers setting under the \[distributedSearch\] stanza. Specify the peers as a set of comma-separated values \(host names or IP addresses with management ports\). For example:

   ```text
   [distributedSearch]
   servers = https://192.168.1.1:8089,https://192.168.1.2:8089
   ```

   Note: You must precede the host name or IP address with the URI scheme, either "http" or "https".

3. Restart the search head.

**Distributing key files**

If you add search peers via Splunk Web or the CLI, Splunk Enterprise automatically configures authentication. However, if you add peers by editing distsearch.conf, you must distribute the key files manually. After adding the search peers and restarting the search head, as described above:

* Copy the file ```$SPLUNK_HOME/etc/auth/distServerKeys/trusted.pem`` from the search head to``` $SPLUNK\_HOME/etc/auth/distServerKeys//trusted.pem\`\`\` on each search peer.

  The is the search head's serverName, specified in server.conf.

* Restart each search peer.

**Authentication of multiple search heads from a single peer**

Multiple search heads can search across a single peer. The peer must store a copy of each search head's certificate.

The search peer stores the search head keys in directories with the specification `$SPLUNK_HOME/etc/auth/distServerKeys/`.

For example, if you have two search heads, named A and B, and they both need to search one particular search peer, do the following:

1. On the search peer, create the directories `$SPLUNK_HOME/etc/auth/distServerKeys/A/` and `$SPLUNK_HOME/etc/auth/distServerKeys/B/`.
2. Copy A's trusted.pem file to `$SPLUNK_HOME/etc/auth/distServerKeys/A/` and B's trusted.pem to `$SPLUNK_HOME/etc/auth/distServerKeys/B/`.
3. Restart the search peer.

**Forward internal logs to the search peers**

Create an outputs.conf file that configures the search heads for load-balanced forwarding across the set of search peers \(indexers\). You must also turn off indexing so they do not both retain the data locally as well as forward it to the search peers.

[Forward internal data to search peers]()

Note: You perform the same configuration steps to forward data from search head cluster members to their set of search peers. _However, you must ensure that all members use the same outputs.conf file_. To do so, do not edit the file on the individual search heads. Instead, _use a deployer to propagate the file across the search head cluster_.

**Integrate with a single-site indexer cluster**

Configure each search head cluster member as a search head on the indexer cluster. Use the CLI splunk edit cluster-config command. For example:

`splunk edit cluster-config -mode searchhead -master_uri https://10.152.31.202:8089 -secret newsecret123`

`splunk restart`

This example specifies:

* The instance is a search head in an indexer cluster.
* The master node of the indexer cluster resides at 10.152.31.202:8089.
* The secret key is "newsecret123". You must use the same secret key across all nodes in both the indexer cluster and the search head cluster.

You must do this for each member of the search head cluster.

**Integrate with a multisite indexer cluster**

In a multisite indexer cluster, each search head and indexer has an assigned site. Multisite indexer clustering promotes disaster recovery, because data is allocated across multiple sites. For example, you might configure two sites, one in Boston and another in New York. If one site fails, the data remains accessible through the other site.

To integrate search head cluster members with a multisite indexer cluster, configure each member as a search head on the indexer cluster, as in the single-site example. The only difference is that you must also specify the site for each member. This should ordinarily be "site0", so that all search heads in the cluster perform their searches across the same set of indexers. For example:

`splunk edit cluster-config -mode searchhead -site site0 -master_uri https://10.152.31.202:8089 -secret newsecret123`

`splunk restart`

```text
[shclustering]
conf_deploy_fetch_url = https://:8089
disabled = 0
mgmt_uri = https://fldcvpsla9448.wdw.disney.com:8089
pass4SymmKey = 
replication_factor = 2
id = 

[clustering]
master_uri = https://:8089
mode = searchhead
multisite = true
pass4SymmKey = 
```

#### Final Steps

* Add users to the search heads
* Install a load balancer in front of the search heads. This step is optional. See "Use a load balancer with search head clustering."
* Use the deployer to distribute apps and configuration updates to the search heads.

[top]()

## Create a Deployment Server

To set up a deployment server, you need to configure both the deployment server and the deployment clients, although most configuration occurs on the deployment server side. The main actions you need to perform are these:

* Configure the deployment clients to connect to a deployment server \(Universal Forwarders, etc.\)
* Create directories on the deployment server to hold the deployment apps and populate them with content \(inputs.conf, etc.\)
* Create mappings between deployment clients and app directories \(the server classes\)\(serverclass.conf\)

Notes: You can use a deployment server to distribute updates to search heads in indexer clusters, as long as they are standalone search heads. You cannot use the deployment server to distribute updates to members of a search head cluster - you must use a Deployer.

Because of high CPU and memory usage during app downloads, it is recommended that the deployment server instance reside on a dedicated machine.

### Create Deployment apps

A deployment app consists of any arbitrary content that you want to download to a set of deployment clients. The content can include:

* A Splunk Enterprise app \(such as those on Splunkbase\)
* A set of Splunk Enterprise configurations
* Other content, such as scripts, images, and supporting files

You add a deployment app by creating a directory for it on the deployment server.

You create separate directories for each deployment app in a special location on the deployment server. The default location is `$SPLUNK_HOME/etc/deployment-apps`, but this is configurable through the repositoryLocation attribute in serverclass.conf. Underneath this location, each app must have its own subdirectory. The name of the subdirectory serves as the app name in the forwarder management interface. By creating an app directory, you have effectively created the app itself, even if the directory does not yet contain any content.

After creating any new app directories, you must run the CLI reload deploy-server command to make the deployment server aware of them: `splunk reload deploy-server`

Note: After an app is downloaded, it resides under `$SPLUNK_HOME/etc/apps` on the deployment clients.

An example of how to create an app and its settings that can be deployed from a Deployment Server is:

1. Create an app-name subdirectory under \`\`\`/opt/splunk/etc/deployment-apps/\`\`
2. Create a /local directory under that subdirectory
3. In the /local directory, create an **inputs.conf** file to specify the inputs

For example, `/opt/splunk/etc/deployment-apps/MyApp/local/inputs.conf` might contain:

```text
[monitor:///var/log/nodejs*/*/error*.log]
index = myapp
sourcetype = app_err
ignoreOlderThan = 30d

[monitor:///var/log/nodejs*/*/out*.log]
index = myapp
sourcetype = app_out
ignoreOlderThan = 30d
```

These entries define:

* The log file to be monitored
* The index to put the log data into
* The sourcetype to apply to this input
* An entry to tell Splunk not to ingest any log entries older than 30 days

See [inputs.conf.example](https://github.com/packetiq/SplunkArchitect/blob/master/README/inputs.conf.example) and [inputs.conf.spec](https://github.com/packetiq/SplunkArchitect/blob/master/README/inputs.conf.spec) for more option settings.

### Define a server class

A **server class** is a mapping between deployment clients and apps. It tells the deployment server which apps to send to which clients. Therefore, when you define a server class, you associate one or more apps with a group of deployment clients, with these steps:

1. Create the server class.
2. Specify one or more deployment apps for the server class.
3. Specify the clients that belong to the server class.

This info gets saved in a serverclass.conf file in `/opt/splunk/etc/system/local/`, although you could also have: `/opt/splunk/etc/apps/SomeApp/local/serverclass.conf` in which case the settings would be merged by standard Splunk precedence.

```text
[serverClass:app_name]
# Set the attribute to ipAddress, hostname, DNSname, or clientName
whitelist.0 = servername(0001|0002|0003).yourfqdn.com
whitelist.1 = servername(0027|0028|0030).yourfqdn.com
whitelist.2=*.fflanda.com

blacklist.0=printer.fflanda.com
blacklist.1=scanner.fflanda.com

restartSplunkd = true
[serverClass:app_name:app:app_directory]
[serverClass:app_name:app:wdpr_forwarderfullspeed]

# There is also a machineTypesFilter which will match any machine types 
# in a comma-delimited list. Common machine types are: 
# linux-x86_64, windows-x64, linux-i686, 
# freebsd-i386, darwin-i386, sunos-sun4u, 
# linux-x86_64, sunos-i86pc, freebsd-amd64.

```

You can configure the serverclass.conf file directly, or use Splunk Web:

* Settings &gt; Forwarder management
* Server Classes \(tab\) &gt; New Server class  Note: Server class names must be unique.
* Add Apps \(tab\)
* Add Clients \(tab\)

**Deploy Apps**

When you create a server class, you map a set of clients to a set of apps. After you specify both the client filters and the apps, the deployment server automatically deploys the apps to the qualifying clients.

After you edit the content of an app, you must reload the deployment server so that the deployment server learns of the changed app. It then redeploys the app to the mapped set of clients.

To reload the deployment server, use the CLI `reload deploy-server`

### Configure the deployment client

The deployment client \(such as Universal Forwarders\)

On the deployment client \(such as a Universal Forwarder\), run these CLI commands:

`splunk set deploy-poll:` `splunk restart`

Use the IP\_address/hostname and management\_port of the deployment server you want the client to connect with.

You can also directly create and edit a `deploymentclient.conf` file in `$SPLUNK_HOME/etc/system/local/`

The deploymentclient.conf file requires two stanzas:

\[deployment-client\] Configures a number of attributes, including where to find new or updated content. You do not usually need to change the default values for this stanza.

\[target-broker:deploymentServer\] Specifies the location of this client's deployment server. deploymentServer is the default name for a deployment server. You must specify the deployment server under this stanza.

```text
[deployment-client]
# Set the phoneHome period to 10 minutes (default is 30 seconds)
phoneHomeIntervalInSecs = 600
# optional clientName to display in the Deployment Server 
# clientName = 

[target-broker:deploymentServer]
# set this to the Deployment Server url
targetUri = :8089
```

**Get deployment client information**

You can find information about the deployment client from two locations:

* On the deployment client itself: Settings &gt; Server settings &gt; Deployment client settings
* On the deployment server: Settings &gt; Forwarder management

To disable a deployment client: `splunk disable deploy-client`

[Forward internal data to search peers]()

[top]()

## Create a Deployer

Deployer functionality is only for use with search head clustering, but it is built into all Splunk Enterprise instances running version 6.2 or above. The processing requirements for a deployer are fairly light, so you can usually co-locate deployer functionality on an instance performing some other function. You have several options as to the instance on which you run the deployer:

* If you have a deployment server that is servicing only a small number of deployment clients \(no more than 50\), you can run the deployer on the same instance as the deployment server. The deployer and deployment server functionalities can interfere with each other at larger client counts. See Deployment server provisioning in Updating Splunk Enterprise Instances.
* If you are running an indexer cluster, you might be able to run the deployer on the same instance as the indexer cluster master node. Whether this option is available to you depends on the master's load. See Additional roles for the master node in Managing Indexers and Clusters of Indexers for information on cluster master load limits.
* If you have a monitoring console, you can run the deployer on the same instance as the console. See Which instance should host the console? in Monitoring Splunk Enteprise.
* If none of the instance types enumerated above are available, run the deployer on a dedicated Splunk Enterprise instance.

A deployer can service only a single search head cluster. If you have multiple clusters, you must use a separate deployer for each one. The deployers must run on separate instances.

**Important:**

* Do not locate deployer functionality on a search head cluster member. The deployer must run on a separate instance from any cluster member.
* Do not use a deployment server to update search head cluster members; you must use a search head cluster deployer.

**Creating a Deployer**

_**Deployer functionality is automatically enabled on all Splunk Enterprise instances**_**.** The main configuration step is to specify the deployer's security key, as described in the next step. Later in the deployment process, you point the cluster members at this deployer instance, so that they have access to it.

**Configure the deployer's security key**

The deployer uses the security key to authenticate communication with the cluster members. The cluster members also use it to authenticate with each other. You must set the key to the same value on all cluster members and the deployer. You set the key on the cluster members when you initialize them.

To set the key on the deployer, specify the pass4SymmKey attribute in either the \[general\] or the \[shclustering\] stanza of the deployer's server.conf file. For example:

```text
[shclustering]
pass4SymmKey = yoursecuritykey
```

**Set the search head cluster label on the deployer.**

The search head cluster label is useful for identifying the cluster in the monitoring console. This parameter is optional, but if you configure it on one member, you must configure it with the same value on all members, as well as on the deployer.

To set the label, specify the shcluster\_label attribute in the \[shclustering\] stanza of the deployer's server.conf file. For example:

```text
[shclustering]
shcluster_label = shcluster1
```

To be completed...

* Install apps on the Deployer & deploy to the search head cluster
* Directories on the Deployer to put apps & knowledge objects in
* Where those apps & knowlege objects end up on the search heads

[Forward internal data to search peers]()

[top]()

## Create a License Server

To be completed...

[Forward internal data to search peers]()

#### Licenses for distributed search

Each instance in a distributed search deployment must have access to a license pool. This is true for both search heads and search peers. See Licenses for search heads in Admin Manual.

[top]()

## Install and Configure a Universal Forwarder

**Forwarding data to an indexer** To forward remote data to an indexer, you use forwarders, which are Splunk Enterprise instances that receive data inputs and then consolidate and send the data to a Splunk Enterprise indexer.

Forwarders come in two flavors:

* **Universal forwarders**. These are a separate installation package that maintains a small footprint on a host machine. They perform minimal processing on the incoming data streams before forwarding them on to an indexer, also known as the receiver.
* **Heavy forwarders**. These are a full installation of Splunk Enterprise and retain most of the basic functionality. They can parse data before forwarding it to the receiving indexer, store indexed data locally, and/or forward the parsed data to a receiver for final indexing on that machine as well.

Both types of forwarders tag data with metadata such as host, source, and source type before forwarding it on to the indexer.

Forwarders allow you to use resources efficiently when processing large quantities or disparate types of data coming from remote sources. They also offer capabilities for load balancing, data filtering, and routing.

**Manual Installation**

* Access the Splunk website:  [Download Splunk Universal Forwarder](https://www.splunk.com/en_us/download/universal-forwarder.html#tabs/linux)

**Note: You will have to create an account and/or login using your Splunk user ID & password.**

* Select and download the appropriate rpm package \(linux &gt; 64-bit &gt; .rpm\)
* On the next Splunk web page \(that shows up after you select a download\), click the 'Download via Command Line \(wget\)' link and copy the string - for example:

```text
wget -O splunkforwarder-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm 'https://www.splunk.com/bin/splunk/DownloadActivityServlet?architecture=x86_64&platform=linux&version=6.5.3&product=universalforwarder&filename=splunkforwarder-6.5.3-36937ad027d4-linux-2.6-x86_64.rpm&wget=true'
```

Or...

* Use ftp or WinSCP to copy the package to the splunk server.
* `sudo su` \(root\)
* Install the package `rpm -i splunkforwarder-<…>-linux-2.6-x86_64.rpm`
* Start the Forwarder: `cd /opt/splunkforwarder/bin/` `./splunk start`

**Remote / Automated Installation** You can craft an installation script using the following links as starting points, and leveraging the **wget** command:

[docs.splunk.com/Documentation/Forwarder/6.5.3/Forwarder/Installanixuniversalforwarderremotelywithastaticconfiguration](http://docs.splunk.com/Documentation/Forwarder/6.5.3/Forwarder/Installanixuniversalforwarderremotelywithastaticconfiguration) [answers.splunk.com/answers/34896/simple-installation-script-for-universal-forwarder](https://answers.splunk.com/answers/34896/simple-installation-script-for-universal-forwarder.html) [answers.splunk.com/answers/100989/forwarder-installation-script](https://answers.splunk.com/answers/100989/forwarder-installation-script)

For most types of cluster deployments, you should enable indexer acknowledgment for the forwarders sending data to the peer.

You can simplify the process of connecting forwarders to peer nodes by using the indexer discovery feature.

### Configuring inputs.conf

### Configuring outputs.conf

The universal forwarder ships with these default versions of outputs.conf:

* One in `$SPLUNK_HOME/etc/system/default`.
* Another in `$SPLUNK_HOME/etc/apps/SplunkUniversalForwarder/default`.

The default version in the SplunkUniversalForwarder app has precedence over the version under /etc/system/default. Do not edit default versions of any configuration files.

[top]()

## Splunk Common Network Ports

The image above was obtained from [Job Jordan's Blog:](http://jordan2000.com/splunk-common-network-ports/) [http://jordan2000.com/splunk-common-network-ports](http://jordan2000.com/splunk-common-network-ports)

Other images in this document were obtained and/or modified from images copied from Splunk documentation. Splunk is not responsible for any inaccuracies introduced by my modifications.

[top]()

> Written with [StackEdit](https://stackedit.io/).

