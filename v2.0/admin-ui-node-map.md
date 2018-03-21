---
title: Node Map 
toc: false
---

The **Node Map** view visualizes the geographical configuration of a multi-regional cluster by plotting the node locations on a world map. The **Node Map** also provides real-time cluster metrics, with the ability to drill down to individual nodes to monitor and troubleshoot the cluster health and performance. 

<div id="toc"></div>

{{site.data.alerts.callout_info}}The <b>Node Map</b> is an <a href="enterprise-licensing.html">enterprise-only</a> feature. However, you can <a href="https://www.cockroachlabs.com/pricing/request-a-license/">request a trial license</a>  to try out the <b>Node Map</b> view. {{site.data.alerts.end}}

## Why Use the Node Map View

The **Node Map View** helps you monitor the cluster health and performance, ensure effective resource utilization, and troubleshoot cluster issues:

### Monitor Cluster Health and Performance

The **Node Map** view provides:

- A high-level graphical overview of how the cluster is configured, which helps you understand how the available hardware is servicing your users. 
- Important cluster health metrics, such as Capacity and CPU utilization, QPS, and uptime.

### Ensure Effective Resource Utilization

The **Node Map** view helps you:

- Confirm that the database is balanced and you are utilizing the resource effectively. This helps you provision hardware and plan ahead for scale.
- Ensure that storage capacity is balanced across the disks available to the extent that they match the zone configs.
- Check if there resource congestion anywhere (for example certain nodes going beyond capacity).

### Troubleshoot the Cluster

The **Node Map** view helps you identify and preempt any scenarios that might cause downtime, service issues, or otherwise negatively impact cluster health. The **Node Map** view helps you:

- Monitor disk utilization to know if you need to add more storage capacity.
- Identify dead nodes in the cluster.
- Be informed about unavailable or under-replicated ranges.
- Confirm that configuration changes were persisted to the cluster.
- Observe QPS across nodes to ensure that queries are rerouted to live nodes if a node failure occurs.

## Configure and Navigate the Node Map

To configure the **Node Map**, you need to start the cluster with the correct `--localities` flags and assign the latitudes and longitudes for each region.

{{site.data.alerts.callout_info}}The <b>Node Map</b> won't be displayed until all regions are assigned the corresponding latitudes and longitudes. {{site.data.alerts.end}}

Consider a four-node geo-distributed cluster with the following configuration:

|  Node | Region | Datacenter |
|  ------ | ------ | ------ |
|  N1 | us-east | us-east-1 |
|  N2 | us-east | us-east-1 |
|  N3 | us-west | us-west-1 |
|  N4 | eu-west | eu-west-1 |

#### Step 1. Ensure the CockroachDB version is 2.0 or higher

~~~ shell
$ ./cockroach version
~~~

If not, [upgrade to CockroachDB v2.0](upgrade-cockroach-version.html).

#### Step 2. Start the nodes with the correct `--localities` flags

In a new terminal, start Node 1:

{% include copy-clipboard.html %}
~~~ shell
./cockroach start \
--insecure \
--locality=region=us-east,datacenter=us-east-1  \
--store=node1 \
--host=localhost \
--port=26257 \
--http-port=8080 \
--join=localhost:26257,localhost:26258,localhost:26259
~~~

In a new terminal, start Node 2:

{% include copy-clipboard.html %}
~~~ shell
./cockroach start \
--insecure \
--locality=region=us-east,datacenter=us-east-1 \
--store=node2 \
--host=localhost \
--port=26258 \
--http-port=8081 \
--join=localhost:26257,localhost:26258,localhost:26259
~~~

In a new terminal, start Node 3:

{% include copy-clipboard.html %}
~~~ shell
./cockroach start \
--insecure \
--locality=region=us-west,datacenter=us-west-1 \
--store=node3 \
--host=localhost \
--port=26259 \
--http-port=8082 \
--join=localhost:26257,localhost:26258,localhost:26259
~~~

In a new terminal, start Node 4:

{% include copy-clipboard.html %}
~~~ shell
./cockroach start \
--insecure \
--locality=region=eu-west,datacenter=eu-west-1 \
--store=node4 \
--host=localhost \
--port=26260 \
--http-port=8083 \
--join=localhost:26257,localhost:26258,localhost:26259
~~~

In a new terminal, use the [`cockroach init`](initialize-a-cluster.html) command to perform a one-time initialization of the cluster:

{% include copy-clipboard.html %}
~~~ shell
./cockroach init --insecure
~~~

#### Step 3. [Set the enterprise license](enterprise-licensing.html)

#### Step 4. Insert the latitudes and longitudes of each region into the `systems.locations` table

{% include copy-clipboard.html %}
~~~ sql
> INSERT INTO system.locations VALUES
  ('region', 'us-east', 40.367474, -82.996216), 
  ('region', 'us-west', 43.8041334, -120.55420119999997), 
  ('region', 'eu-west', 48.856614, 2.3522219000000177);
~~~

{{site.data.alerts.callout_info}}The <b>Node Map</b> won't be displayed until all regions are assigned the corresponding latitudes and longitudes. {{site.data.alerts.end}}

To get the latitudes and longitudes of AWS regions, see [Locations Co-ordinates for Reference](#location-co-ordinates-for-reference).

#### Step 5. Verify the settings 

Navigate to `https://localhost:8080/#/reports/localities` and use the **Localities** table to verify that the latitudes and longitudes correspond to the correct regions.

#### Step 6. View the Node Map

[Navigate to the **Overview page**](admin-ui-cluster-overview.html) and switch to the **Node Map** view from the drop-down box.

#### Step 7. Navigate the Node Map

Suppose you want to navigate to Node 2, which is in datacenter `us-east-1` in the region `us-east`. To navigate to Node 2:

1. Click on map component marked as **region=us-east** on the Node Map. The datacenter view is displayed.
2. Click on the datacenter component marked as **datacenter=us-east-1**. The individual node components are displayed.
3. To navigate back to the cluster view, either click on **Cluster** in the bread-crumb trail at the top of the Node Map, or click **Up to Cluster** in the lower left-hand side of the Node Map.

## Understand the Node Map Components

<img src="{{ 'images/admin-ui-node-map-components.png' | relative_url }}" alt="CockroachDB Admin UI Summary Panel" style="border:1px solid #eee;max-width:90%" />

## Location Co-ordinates for Reference

|  **Location** | **AWS Region** | **GCE Region Name** | **Azure Region** | **Latitude** | **Longitude** |
|  ------ | ------ | ------ | ------ | ------ | ------ |
|  US East (N. Virginia) | us-east-1 | us-east4 | East US | 37.478397 | -76.453077 |
|  US East (Ohio) | us-east-2 |  |  | 40.417287 | -82.907123 |
|  US Central (Iowa) |  | us-central1 | Central US |  |  |
|  US West (N. California) | us-west-1 |  | West US | 38.837522 | -120.895824 |
|  US West (Oregon) | us-west-2 | us-west1 |  | 43.804133 | -120.554201 |
|  Canada (Central) | ca-central-1 | northamerica-northeast1 | Canada Central | 56.130366 | -106.346771 |
|  EU (Frankfurt) | eu-central-1 | europe-west3 | Germany Central | 50.110922 | 8.682127 |
|  EU (Ireland) | eu-west-1 |  | North Europe | 53.142367 | -7.692054 |
|  EU (London) | eu-west-2 | europe-west2 | UK South | 51.507351 | -0.127758 |
|  EU (Paris) | eu-west-3 |  |  | 48.856614 | 2.352222 |
|  Asia Pacific (Tokyo) | ap-northeast-1 | asia-northeast1 | Japan East | 35.689487 | 139.691706 |
|  Asia Pacific (Seoul) | ap-northeast-2 |  | Korea Central | 37.566535 | 126.977969 |
|  Asia Pacific (Osaka-Local) | ap-northeast-3 |  | Japan West | 34.693738 | 135.502165 |
|  Asia Pacific (Singapore) | ap-southeast-1 | asia-southeast1 | Southeast Asia | 1.352083 | 103.819836 |
|  Asia Pacific (Sydney) | ap-southeast-2 | australia-southeast1 |  | -33.86882 | 151.209296 |
|  Asia Pacific (Mumbai) | ap-south-1 | asia-south1 | West India | 19.075984 | 72.877656 |
|  South America (São Paulo) | sa-east-1 | southamerica-east1 | Brazil South | -23.55052 | -46.633309 |