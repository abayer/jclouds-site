---
layout: jclouds
title: Quick Start - HP Cloud Services
---

# Quick Start: HP Cloud Services

This page helps you get started with jclouds API with

1. Sign up for [HP Cloud Services](http://hpcloud.com/).
2. Get your Account ID and Access Key by going to this [page](https://manage.hpcloud.com/api_keys).
3. Ensure you are using a recent version of Java 6.
4. Setup your project to include `hpcloud-objectstorage` and `hpcloud-compute`.
	* Get the dependencies `org.jclouds.provider/hpcloud-objectstorage` and `org.jclouds.provider/hpcloud-compute` using jclouds [Installation]({{ site.url }}/documentation/userguide/installation-guide/index.html).
5. Start coding.

** Note: The identity for hpcloud is the same now as of 1.5.0 which is "tenantName:userName" and the credential is "userPassword"

By default, the authentication mechanism for all OpenStack Keystone based APIs will use your password as the credential to log in.

The following specifications may serve as a guide if you wish to set API Access Keys:
properties.setProperty(KeystoneProperties.CREDENTIAL_TYPE, CredentialTypes.API_ACCESS_KEY_CREDENTIALS)

To get the CredentialTypes class, please see the [Javadoc](http://demobox.github.com/jclouds-maven-site-1.5.0/1.5.0/jclouds-multi/apidocs/org/jclouds/openstack/keystone/v2_0/config/CredentialTypes.html).

## HP Cloud Object Storage

{% highlight java %}
// Get a context with hpcloud that offers the portable BlobStore API
BlobStoreContext context = ContextBuilder.newBuilder("hpcloud-objectstorage")
                 .credentials("tenantName:accessKey", "password")
                 .buildView(BlobStoreContext.class);

// Create a container in the default location
context.getBlobStore().createContainerInLocation(null, container);

// Use the map interface for easy access to put/get things, keySet, etc.
context.createInputStreamMap(container).put("blob.txt", inputStream);

// When you need access to hpcloud specific features, use the provider-specific context
HPCloudObjectStorageClient hpcloudClient = 
	HPCloudObjectStorageClient.class.cast(context.getProviderSpecificContext().getApi());

// Create a container with public access
boolean accessibleContainer = hpcloudClient.createContainer("public-container", withPublicAccess());

ContainerMetadata cm = hpcloudClient.getContainerMetadata("public-container");
if (cm.isPublic()) {
	...
}

// When you want to use CDN features with a container, use the provider-specific CDN client
HPCloudCDNClient cdnClient = hpcloudClient.getCDNExtension().get();

// Get a CDN URL for the container
URI uri = cdnClient.enableCDN(container);

// Get the CDN Metadata for the container
ContainerCDNMetadata cdnMetadata = cdnClient.getCDNMetadata(container)
if (cdnMetadata.isCDNEnabled()) {
    ...
}

// Be sure to close the context when done
context.close();
{% endhighlight %}

## HP Cloud Compute

{% highlight java %}
// Get a context with hpcloud-compute that offers the portable ComputeService API
ComputeServiceContext ctx = ContextBuilder.newBuilder("hpcloud-compute")
                      .credentials("tenantName:accessKey", "password")
                      .modules(ImmutableSet.<Module> of(new Log4JLoggingModule(),
                                                        new SshjSshClientModule()))
                      .buildView(ComputeServiceContext.class);

ComputeService cs = ctx.getComputeService();

// List availability zones
Set<? extends Location> locations = cs.listAssignableLocations();

// List nodes
Set<? extends ComputeMetadata> nodes = cs.listNodes();

// List hardware profiles
Set<? extends Hardware> hardware = cs.listHardwareProfiles();

// List images
Set<? extends org.jclouds.compute.domain.Image> image  = cs.listImages();

// Create nodes with templates
Template template = cs.templateBuilder().osFamily(OsFamily.UBUNTU).build();
Set<? extends NodeMetadata> groupedNodes = cs.createNodesInGroup("myGroup", 2, template);

// Reboot images in a group
cs.rebootNodesMatching(inGroup("myGroup"));

// When you need access to HP Cloud Compute features, use the provider-specific context
RestContext<NovaClient, NovaAsyncClient> context = ctx.getProviderSpecificContext();
NovaClient client = context.getApi();

// From the provider-specific context, you can access servers, flavors, and images through their respective clients
// Get the server client for a particular availability zone
ServerClient serverClient = client.getServerClientForZone(zone);

// List all available servers for an account
Set<Server> servers = serverClient.listServersInDetail()

// Get the flavor client for a particular availability zone
FlavorClient flavorClient = client.getFlavorClientForZone(zone);

// List all available flavors
Set<Flavor>flavors = flavorClient.listFlavorsInDetail();

// Get the details of a particular flavor
Flavor flavor = flavorClient.getFlavor(flavorId);

// Get the image client for a particular availability zone
ImageClient imageClient = client.getImageClientForZone(zone);

// List all available images
Set<Image> images = imageClient.listImagesInDetail();

// Get the details of a particular image
Image image = imageClient.getImage(images.iterator().next().getId());

// From the provider-specific context, you can also retrieve the optional extensions related to key pair, security
// group, and floating IP address management
// Get the optional keypair client for a particular availability zone
KeyPairClient keyPairClient = client.getKeyPairExtensionForZone(zone).get();

// Create a new keypair
KeyPair keyPair = keyPairClient.createKeyPair("exampleKeyPair");

// Get the optional security groups client for a particular availability zone
SecurityGroupClient securityGroupClient = client.getSecurityGroupExtensionForZone(zone).get();

// List all available security groups
Set<SecurityGroup> securityGroups = securityGroupClient.listSecurityGroups();

// Create a new security group
SecurityGroup exampleSecurityGroup = securityGroupClient
    .createSecurityGroupWithNameAndDescription("exampleSecurityGroup", "an example security group");

// Create a rule for an existing security group
Ingress ingress = Ingress.builder().ipProtocol(IpProtocol.TCP).fromPort(80).toPort(8080).build();
SecurityGroupRule rule = securityGroupClient
        .createSecurityGroupRuleAllowingSecurityGroupId(exampleSecurityGroup.getId(), ingress, "0.0.0.0/0");

// Get the optional floating IP client for a particular availability zone
FloatingIPClient floatingIPClient = client.getFloatingIPExtensionForZone(zone).get();

// List all available floating ip addresses
Set<FloatingIP> addresses = floatingIPClient.listFloatingIPs();

// Create/allocate a new address
FloatingIP exampleAddress = floatingIPClient.allocate();

// Associate a server to an existing address
floatingIPClient.addFloatingIPToServer(exampleAddress.getIp(), server.getId());

// Be sure to close the context when done
context.close();
{% endhighlight %}
