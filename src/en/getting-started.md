Title: Getting started with Juju

# Getting started with Juju 

Juju works across public clouds, private clouds, and locally with LXD-based
deployments. Juju works through a controller that will process all of the
events that occur. These instructions will walk you through using the hosted
Juju as a Service (JAAS) controller and requires you to have public cloud
credentials ready. If you'd prefer to test it out locally, see the [Getting
started with Juju and LXD][tut-lxd] guide.

Apart from Juju, the following technologies will be used:
   
- [LXD][LXD-upstream]: a hypervisor for LXC, providing fast, secure containers.
- [ZFS][ZFS-wiki]: a highly efficient and feature-rich filesystem and logical volume manager.


## Install the software

Begin by installing the required software:

JAAS uses your [Ubuntu SSO][ubuntuSSO] account for authentication - if you
don't yet have an SSO account you can sign up for one [here][ubuntuSSO] (it's
easy and free).

## Create a Model

<style>
table th, table td {
    background: #f7f7f7;
    border: 0px solid;
    padding: 15px 10px;
}

table.logos {
    background: #f7f7f7;
    border: 0px solid;
    padding: 4px 4px;
}

table.logos th, table.logos td{
    align="center";
    valign="center";
    border: 8px;
    border-style: solid;
    border-color: #ffffff;
  }
</style>

<table width="500" border-width="0px" cellpadding="5">

<tr>

<td align="center" valign="center" border-width="0px" >
<img src="./media/jaas-login-1.png" alt="Login to JAAS" />
<br />
Press the green Start building button to get started...
</td>

<td align="center" valign="center" border-width="0px">
<img src="./media/jaas-login-2.png" alt="Login to JAAS" />
<br />
... and reveal your (currently empty) model.
</td>

</tr>

</table>


You build your [model][models] by adding and relating applications from the
[Charm Store][charmstore]. Press the green '+' symbol in the middle of the UI
to start finding applications in the Charm Store.

You will see a list of available charms and bundles with a description of
each. Select a [charm][charms] or [bundle][bundles] to learn more about it.

When you have selected a charm or bundle it can be added to your model by
pressing the 'Add to model' button...

Hint: to see how easy JAAS makes complex deployments, try the [Canonical
Kubernetes bundle][k8]

<table width="500" border-width="0px" cellpadding="5">

<tr>

<td align="center" valign="center" border-width="0px" >
<img src="./media/jaas-kubernetes.png" alt="Canonical kubernetes bundle" />
<br />
Press the green Add to model button to select the solution.
</td>

<td align="center" valign="center" border-width="0px">
<img src="./media/jaas-deploy-changes.png" alt="deploying changes in JAAS" />
<br />
Press the blue Deploy changes button to proceed with the changes you've made
with your model.
</td>

</tr>

</table>

After you press `Deploy changes` you can adjust your model name and choose a
cloud. When you select a cloud, you will be guided through
the process of entering credentials for it.

## Prepare your cloud credentials

We recommend that you use each public cloud's identity and access management
(IAM) tool to generate a new set of credentials exclusively for use with JAAS.

See [Cloud credentials][credentials] for more information, along with the
following links for your specific cloud:

- [Amazon Web Services][aws-creds]
- [Microsoft Azure][azure-creds]
- [Google Compute Engine][gce-creds].

## Deploy

Click on `Deploy` to confirm your cloud information and build your model.

Deploying takes a few minutes - in this period, instances are being
created in the cloud, software is being installed, applications are being
related to each other and configuration is being applied.

<table width="500" border-width="0px" cellpadding="5">

<tr>

<td align="center" valign="center" border-width="0px" >
<img src="./media/jaas-deploy-1.png" alt="description here" />
<br />
Deploying a bundle can take a few minutes...
</td>

<td align="center" valign="center" border-width="0px">
<img src="./media/jaas-deploy-2.png" alt="description here" />
<br />
When complete, the application icons will turn grey.
</td>

</tr>

</table>



As the applications become operational, the colours in the model view will
change to grey to indicate an idle state, and the pending notices in the
inspector on the left will disappear to show that everything is working as
expected.

!!! Tip: You can check and manage existing models through the JAAS GUI by
clicking on your username

## Use the command line

Juju can also be used from the command line. Models you've created with the
JAAS website can also be operated via the normal Juju command line. In order
to use the command line, you will first need to install the Juju client
software on your machine.

Juju is available for various types of Linux, macOS, and Windows.
Click on the sections below for the relevant instructions or visit the
[install instructions][install] for a thorough run down of your options.

Get started quickly by picking your OS from below:

<style>
details  {
    padding-bottom: 6px;
}
</style>

^# On an OS which supports snap packages

   For Ubuntu 16.04 LTS (Xenial) and other OSes which have snaps enabled
   ([see the snapcraft.io site][snapcraft]) you can install the latest
   stable release of Juju with the command:

         sudo snap install juju --classic

   You can check which version of Juju you have installed with:

## Groups and LXD initialisation 

Firstly, in order to use LXD, your user must be a member of the `lxd` group.
This should already be the case but you can confirm this by running the
command:

```bash
groups
```

Your groups may vary, but if `lxd` is absent you can add yourself to the `lxd`
group with:

```bash
newgrp lxd
```

Secondly, LXD includes an interactive initialisation which includes setting up
a ZFS pool and appropriate networking for your LXD containers. To start this
process, enter:

```bash
sudo lxd init
```

You will be asked several questions to configure LXD for use. Pressing Enter
will accept the default answer (provided in square brackets). Only one answer,
the size of the loop device in the below example, uses a non-default value.

```no-highlight
Name of the storage backend to use (dir or zfs) [default=zfs]: 
Create a new ZFS pool (yes/no) [default=yes]? 
Name of the new ZFS pool [default=lxd]:
Would you like to use an existing block device (yes/no) [default=no]? 
Size in GB of the new loop device (1GB minimum) [default=10GB]: 20
Would you like LXD to be available over the network (yes/no) [default=no]? 
Do you want to configure the LXD bridge (yes/no) [default=yes]?
```

The bridge network will then be configured via a second round of questions.
Except in the case where the randomly chosen subnet may conflict with an
existing one in your local environment, it is fine to accept all the default
answers. IPv6 networking (the last question) is not required for Juju.

^# Walkthrough of network configuration  

   In order for networking to be established between containers and Juju, you 
   need to set up a bridge device.

   !["step 1"](./media/juju-lxd-config001.png)

   The default name for the bridge device is `lxdbr0`. This name _must_ be used 
   for Juju to be able to connect to the containers.
   
   !["step 2"](./media/juju-lxd-config002.png)
   
   Juju will expect an IPv4 network space for the containers, so you should 
   enable this.
   
   !["step 3"](./media/juju-lxd-config003.png)
   
   The default address is chosen randomly in the 10.x.x.x space. You do not 
   need to change this unless it conflicts with another subnet you know is on
   your network.
   
   !["step 4"](./media/juju-lxd-config004.png)
   
   You need to enter a [CIDR](https://tools.ietf.org/html/rfc4632) mask value. 
   The default of 24 gives you a possible 254 addresses for the subnet.
   
   !["step 5"](./media/juju-lxd-config005.png)
   
   You can now specify the start of the DHCP address range...
   
   !["step 6"](./media/juju-lxd-config006.png)
   
   And the end address of the range...
   
   !["step 7"](./media/juju-lxd-config007.png)
   
   You can also specify the total number of DHCP clients to accept.
   
   !["step 8"](./media/juju-lxd-config008.png)
   
   Finally for IPv4, enable Network Address Translation (NAT) to allow the
   containers to communicate with the outside world.
   
   !["step 9"](./media/juju-lxd-config009.png)
   
   You can continue to set up a similar IPv6 bridge device, but this is not 
   necessary for Juju.
   
   !["step 10"](./media/juju-lxd-config010.png)
   
LXD is now configured to work with Juju.

!!! Note: LXD adds iptables (firewall) rules to allow traffic to the
subnet/bridge it created. If you subsequently add/change firewall settings
(e.g. with `ufw`), ensure that such changes have not interfered with Juju's
ability to communicate with LXD.

## Create a controller

Before you can start deploying applications, Juju needs to bootstrap a
controller for the LXD configuration we created. The controller manages
both the environment and the models you create to host the applications.

The `juju bootstrap` command is used to create the controller. The command
expects a name (for referencing this controller) and a cloud to use. The LXD
'cloud' is known as 'localhost' to Juju.

For our LXD localhost cloud, we will create a controller called 'lxd-test':

```bash
juju bootstrap localhost lxd-test
```

This may take a few minutes as LXD must download an image for Xenial. A cache
will be used for subsequent containers.

Once the process has completed you can check that the controller has been
created:

```bash
juju controllers 
```

This will return a list of the controllers known to Juju, which at the moment is
the one we just created:
  
```no-highlight
Use --refresh flag with this command to see the latest information.

Controller  Model    User   Access     Cloud/Region         Models  Machines HA  Version
lxd-test*   default  admin  superuser  localhost/localhost       2         1 none  2.0.0
```

A newly-created controller has two models: The 'controller' model, which should
be used only by Juju for internal management, and a 'default' model, which is
ready for actual use.

The following command shows the currently active controller, model, and user:

```bash 
juju whoami
```

In our example, the output should look like this:

```no-highlight
Controller:  lxd-test
Model:       default
User:        admin
```

## Deploy applications

Juju is now ready to deploy applications from among the hundreds included in
the [Juju charm store][charm store]. It is a good idea to test your new model.
How about a MediaWiki site?

```bash
juju deploy cs:bundle/mediawiki-single
```

This will fetch a 'bundle' from the Juju store. A bundle is a pre-packaged set
of applications, in this case 'MediaWiki', and a database to run it 
with. Juju will install both applications and add a relation between them - 
this is part of the magic of Juju: it isn't just about deploying software, Juju 
also knows how to connect it all together.

Installing shouldn't take long. You can check on how far Juju has got by running
the command:
 
```bash
juju status
```

When the applications have been installed, the output of the above command will
look something like this:

![juju status](./media/juju-mediawiki-status.png)

There is a lot of useful information there! The important parts for now are
the 'App' section, which shows that MediaWiki and MySQL are installed, and the
'Unit' section, which shows the IP addresses allocated to each. These addresses
correspond to the subnet we created for LXD earlier on.

When Juju is run on a public cloud, it defaults to being secure - you won't be
able to connect to any applications on a public cloud unless they are
specifically exposed using the `juju expose <application>` command. This
adjusts the relevant firewall controls of the cloud to allow external access.
However, traffic to our local LXD environment is not locked down by default, so
for this example there is no need to perform this step.

From the status output, we can see that the IP address for the MediaWiki
site we have created is 10.154.173.2 Open a browser and enter that address 
to see the site.

!["mediawiki site"](./media/juju-mediawiki-site.png)

Congratulations, you have just deployed an application with Juju!

!!! Note: To remove all the applications in the model you just created, it is 
often quickest to destroy the model with the command 
`juju destroy-model default` and then [create a new model][models].

You can switch focus between your models at any time:

## Next Steps

Now that you have a Juju-powered cloud, it is time to explore the amazing
things you can do with it! 

We suggest you continue your journey by discovering 
[how to add controllers for additional clouds][tut-cloud].

This process may take a few minutes to complete, as Juju releases
the running resources back to the cloud.

## Next steps

Congratulations! You have now deployed a complex workload in the cloud without
hours of looking up config options or wrestling with install scripts!

To discover more about what Juju can do for you, we suggest taking a look at
some of the following pages of documentation:

 - [Creating your own cloud controller][tut-google]
 - [Creating models locally with LXD][tut-lxd]
 - [Share your model with other users][share]
 - [Learn more about charms and bundles][learn]
 - [Find out how to customise and manage models][customise]

[azure]: ./help-azure.html "Using the Microsoft Azure public cloud"
[aws]: ./help-aws.html "Using the Amazon Web Service public cloud"
[bundles]: ./charms-bundles.html "Introduction to bundles"
[k8]: https://jujucharms.com/canonical-kubernetes/
[charms]: ./charms.html "Introduction to charms"
[credentials]: ./credentials.html
[gce]: ./help-google.html "Using the Google Compute Engine public cloud"
[jaascli]: ./jaas-cli.html "Using JAAS from the command line"
[jaaslogin]: https://jujucharms.com/login "JAAS login page"
[models]: ./models.html "Introduction to Juju models"
[ubuntuSSO]: https://login.ubuntu.com/ "Ubuntu single sign on"
[users]: ./users-models.html "Users and models"
[gcedashboard]: https://console.cloud.google.com
[gcecredentials]: https://console.developers.google.com/apis/credentials
[install]: ./reference-install.html
[tut-lxd]: ./tut-lxd.html
[tut-google]: ./tut-google.html
[share]: ./tut-users.html
[learn]: ./charms.html
[customise]: ./models.html
[snapcraft]: https://snapcraft.io
[aws-creds]: ./help-aws.html#credentials
[azure-creds]: ./help-azure.html#credentials
[gce-creds]: ./help-google.html#download-credentials
[charmstore]: https://jujucharms.com/store
