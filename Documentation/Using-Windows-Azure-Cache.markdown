Windows Azure Caching is a distributed memory-based cache service that is part of Windows Azure. Orchard ships with two native providers for Windows Azure Caching, allowing Orchard to use Windows Azure Caching as the cache implementation for both output caching and database caching (NHibernate second-level caching). This topic describes how to configure and enable this functionality.

# Overview

Support for Windows Azure Caching comes in the form of two features in the `Orchard.Azure` module:

	Orchard.Azure.OutputCache:
		Name: Windows Azure Output Cache
		Description: Activates an Orchard output cache provider that targets Windows Azure Cache.
		Dependencies: Orchard.OutputCache
		Category: Performance
	Orchard.Azure.DatabaseCache:
		Name: Windows Azure Database Cache
		Description: Activates an NHibernate second-level cache provider that targets Windows Azure Cache.
		Category: Performance

These two features can be enabled separately or in combination.

They both configure Orchard to use Windows Azure Caching as the cache implementation. This is primarily useful when running Orchard in a server farm configuration. As mentioned above, Windows Azure Caching is a distributed memory-based cache service. Without using some form of distributed cache, caches will become out of sync between the different nodes in your farm.

## Role-based vs Shared Caching

Both features have been designed to work with both [Windows Azure Role-based Caching](http://msdn.microsoft.com/en-us/library/windowsazure/hh914161.aspx) and [Windows Azure Shared Caching](http://msdn.microsoft.com/en-us/library/windowsazure/hh697519.aspx). Role-based Caching can be used only from within a Windows Azure Cloud Service. Shared Caching is called via public HTTP endpoints and can be used from anywhere, irrespective of hosting environment.

The caching providers are configured differently depending on which flavor to use. See the [Configuration reference section][ConfigurationReference] for more information.

> NOTE: In Windows Azure SDK 2.1 for unknown reasons Microsoft decided to include separate versions of the Windows Azure Cache client library, one that works with role-based caching and another (older) version that works with shared caching. This might be because shared caching is being decommissioned and Microsoft didn't want to invest in bringing it up to date to match the newer client library required for role-based caching. Until this discrepancy is resolved you will need to change the references to target Windows Azure Shared Caching.

# Enabling for Windows Azure Cloud Services

When deploying Orchard to a Windows Azure Cloud Service using the `Orchard.Azure.sln` solution, the resulting package is already preconfigured with everything required to enable the caching features. By default the cloud service project is configured for co-located role-based caching with 30% of the role instance memory allowed for cache usage.

The only thing you need to change before deploying is the connection string of the storage account you want to use to maintain the cache cluster's runtime state:

![](../Attachments/Using-Windows-Azure-Cache/storage-account.png)

1. Open `Orchard.Azure.sln`.
2. Navigate to `Orchard.Azure.CloudService`, double click the role `Orchard.Azure.Web` to bring up its property page, and navigate to the *Caching* tab.
3. Configure the connection string of the storage account you want to use.
4. Deploy the cloud service.

After deploying you can simply enable the features *Windows Azure Output Cache* and/or *Windows Azure Database Cache* in the admin dashboard.

## Customizing the configuration

You can customize any of the preconfigured settings according to your requirements. For example you might want to:

* Change the configuration from co-located to dedicated role-based caching by adding a dedicated caching role to the cloud service.
* Change the expiration settings of the preconfigured named caches.
* Use a different set of named caches.
* Use shared caching instead of role-based caching.

This can all be done by reconfiguring the cloud service project.

To reconfigure the expiration settings or set of named caches, use the *Caching* tab of the cloud service project:

![](../Attachments/Using-Windows-Azure-Cache/named-caches.png)

Just remember that you must also reconfigure the provider settings to match whatever cache configuration you create, using the *Settings* tab of the cloud service project:

![](../Attachments/Using-Windows-Azure-Cache/role-configuration.png)

# Enabling for Windows Azure Web Sites

> NOTE: When running Orchard in a Windows Azure Web Site only Shared Caching can be used.

Before the caching features can be enabled you must configure the URL and credentials of the cache clusters you want to use. This can be done either before deploying (in Web.config) or after deploying (in the Windows Azure management portal).

## Configuring cache cluster settings *before* deploying:

1. Open `Orchard.sln`.
2. Navigate to `Orchard.Web` and open the `Web.config` file.
3. For output caching, in the `<appSettings>` element add the following settings:
	* `Orchard.Azure.OutputCache.Hostname`
	* `Orchard.Azure.OutputCache.Port`
	* `Orchard.Azure.OutputCache.AuthorizationToken`
	* `Orchard.Azure.OutputCache.IsSharedCaching`
3. For database caching, in the `<appSettings>` element add the following settings:
	* `Orchard.Azure.DatabaseCache.Hostname`
	* `Orchard.Azure.DatabaseCache.Port`
	* `Orchard.Azure.DatabaseCache.AuthorizationToken`
	* `Orchard.Azure.DatabaseCache.IsSharedCaching`
4. Deploy the web site.

Here's an example configuration:

	<appSettings>
		<!-- Output caching -->
		<add key="Orchard.Azure.OutputCache.Hostname" value="myoutputcache.cache.windows.net" />
		<add key="Orchard.Azure.OutputCache.Port" value="22233" />
		<add key="Orchard.Azure.OutputCache.AuthorizationToken" value="YWNzO...ZXNzY29u" />
		<add key="Orchard.Azure.OutputCache.IsSharedCaching" value="true" />
		<!-- Database caching -->
		<add key="Orchard.Azure.DatabaseCache.Hostname" value="mydatabasecache.cache.windows.net" />
		<add key="Orchard.Azure.DatabaseCache.Port" value="22233" />
		<add key="Orchard.Azure.DatabaseCache.AuthorizationToken" value="FQdOS...ImNXFtSJ" />
		<add key="Orchard.Azure.DatabaseCache.IsSharedCaching" value="true" />
	</appSettings>

Refer to the [Configuration reference section][ConfigurationReference] for more details on how to configure each setting.

You can now enable the features *Windows Azure Output Cache* and/or *Windows Azure Database Cache* in the admin dashboard.

## Configuring cache cluster settings *after* deploying:

1. Deploy the web site.
2. In the management portal, navigate to your web site and select the *Configure* tab.
3. For output caching, under *App settings* add the following settings:
	* `Orchard.Azure.OutputCache.Hostname`
	* `Orchard.Azure.OutputCache.Port`
	* `Orchard.Azure.OutputCache.AuthorizationToken`
	* `Orchard.Azure.OutputCache.IsSharedCaching`
3. For database caching, under *App settings* element add the following settings:
	* `Orchard.Azure.DatabaseCache.Hostname`
	* `Orchard.Azure.DatabaseCache.Port`
	* `Orchard.Azure.DatabaseCache.AuthorizationToken`
	* `Orchard.Azure.DatabaseCache.IsSharedCaching`
4. Click *Save*.

Here's an example configuration:

![](../Attachments/Using-Windows-Azure-Cache/website-configuration.png)

Refer to the [Configuration reference section][ConfigurationReference] for more details on how to configure each setting.

You can now enable the features *Windows Azure Output Cache* and/or *Windows Azure Database Cache* in the admin dashboard.

# Enabling for any other hosting

To enable the features when running Orchard in any other hosting environment, use the `Web.config` method described above. Once the appropriate settings have been added to the `<appSettings>` element, can enable the features *Windows Azure Output Cache* and/or *Windows Azure Database Cache* in the admin dashboard.

[ConfigurationReference]: #ConfigurationReference
# Configuration reference

Both Windows Azure Output Cache and Windows Azure Database Cache share the same configuration structure, differing only by a prefix added to each setting. The prefix is `Orchard.Azure.OutputCache.` or `Orchard.Azure.DatabaseCache.` respectively.

These settings are read from platform configuration using the [CloudConfigurationManager](http://msdn.microsoft.com/en-us/library/microsoft.windowsazure.cloudconfigurationmanager.aspx) class, which means they can be configured either in cloud service role configuration settings, web site configuration settings or the `<appSettings>` element in Web.config.

<table>
<thead>
<tr>
	<th>Setting</th>
	<th>Description</th>
</tr>
</thead>
<tbody>
<tr>
	<td><code>HostIdentifier</code></td>
	<td>When using role-based caching, set this to the name of the cloud service role hosting the cache cluster. Auto discover will be used to resolve the cache cluster hostname at runtime.</td>
</tr>
<tr>
	<td><code>CacheName</code></td>
	<td>When using role-based caching, set this to the name of a named cache or leave blank to use the <code>default</code> cache.</td>
</tr>
<tr>
	<td><code>Hostname</code></td>
	<td>Set to the hostname of the cache cluster endpoint. Cannot be combined with <code>HostIdentifier</code>.</td>
</tr>
<tr>
	<td><code>Port</code></td>
	<td>Set to the TCP port of the cache cluster endpoint. Cannot be combined with <code>HostIdentifier</code>.</td>
</tr>
<tr>
	<td><code>AuthorizationToken</code></td>
	<td>Set to the authorization token used to connect to the cache cluster endpoint. Cannot be combined with <code>HostIdentifier</code>.</td>
</tr>
<tr>
	<td><code>IsSharedCaching</code></td>
	<td>Set to <code>true</code> if using Windows Azure Shared Caching or <code>false</code> if using role-based caching.</td>
</tr>
</tbody>
</table>

Here's an example configuration using `Web.config`. This configures Windows Azure Output Cache to use a Windows Azure Shared Caching cluster:

	<appSettings>
		<add key="Orchard.Azure.OutputCache.Hostname" value="mycache.cache.windows.net" />
		<add key="Orchard.Azure.OutputCache.Port" value="22233" />
		<add key="Orchard.Azure.OutputCache.AuthorizationToken" value="YWNzO...ZXNzY29u" />
		<add key="Orchard.Azure.OutputCache.IsSharedCaching" value="true" />
	</appSettings>

Here's the default role configuration when deploying to a cloud service:

![](../Attachments/Using-Windows-Azure-Cache/default-configuration.png)

## Multi-tenancy configuration

For multi-tenancy scenarios each setting can optionally be prefixed with a tenant name followed by colon, such as `SomeTenant:Orchard.Azure.OutputCache.Hostname`. Whenever the caching providers read configuration settings they will always first look for a setting specific for the current tenant, and if no such setting exists, fallback to the default non-prefixed setting.

Here's an example configuration with two tenants, both using output caching with Windows Azure Shared Caching, and each using its own cache cluster:

	<appSettings>
		<!-- Settings for Tenant1 -->
		<add key="Tenant1:Orchard.Azure.OutputCache.Hostname" value="mycache1.cache.windows.net" />
		<add key="Tenant1:Orchard.Azure.OutputCache.AuthorizationToken" value="YWNzO...ZXNzY29u" />
		<!-- Settings for Tenant2 -->
		<add key="Tenant2:Orchard.Azure.OutputCache.Hostname" value="mycache2.cache.windows.net" />
		<add key="Tenant2:Orchard.Azure.OutputCache.AuthorizationToken" value="FQdOS...ImNXFtSJ" />
		<!-- Common settings -->
		<add key="Orchard.Azure.OutputCache.Port" value="22233" />
		<add key="Orchard.Azure.OutputCache.IsSharedCaching" value="true" />
	</appSettings>

# Session state caching

In addition to output caching and database caching, session state storage is also sensitive to server farm scenarios, primarily in solutions where there is no session affinity in the load balancer. The load balancer used in Windows Azure Cloud Services is one example of this.

For this reason the web role in the cloud service project (`Orchard.Azure.Web`) is preconfigured through `Web.config` to use an ASP.NET session state provider that targets Windows Azure Cache. While this provider is enabled, all session state is stored in the distributed cache. This means it is safe for the same user to end up at one farm node in one request and another farm node in the next request, without running the risk of losing her state.

> NOTE: This is not necessary when running Orchard in a Windows Azure Web Site as the load balancer here maintains session affinity using cookies.

First a cache client is configured:

	<dataCacheClients>
		<dataCacheClient name="DefaultCacheClient" useLegacyProtocol="false" connectionPool="true" maxConnectionsToServer="20">
			<autoDiscover isEnabled="true" identifier="Orchard.Azure.Web" />
		</dataCacheClient>
	</dataCacheClients>

Next, session state is configured to use the Azure Cache provider and the cache client defined above:

	<sessionState mode="Custom" timeout="60" customProvider="CacheSessionStateProvider">
		<providers>
			<!-- Set the cacheName attribute to be the name of the configured logical cache to use for session state storage. -->
			<add name="CacheSessionStateProvider" type="Microsoft.Web.DistributedCache.DistributedCacheSessionStateStoreProvider, Microsoft.Web.DistributedCache" cacheName="SessionStateCache" dataCacheClientName="DefaultCacheClient" applicationName="Orchard" />
		</providers>
	</sessionState>

The configuration of the session state provider can be customized by editing `Web.config` before deploying the cloud service. To disable the session state provider, simply comment out the above sections from `Web.config` before deploying the cloud service.