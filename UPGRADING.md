4.1.X to 4.2
============

Process
-------

Ensure that you are on the latest 4.1 series of cookbooks.  We are not
heavily testing upgrades from previous releases directly to havana.

Use knife to edit the environment for the nova cluster.  Ensure that
in override_attributes, there is the following section:

"osops": { "do_package_upgrades": true }

knife cookbook upload the latest cookbooks.
knife role from_file the latest roles.

If this is a quantum configuration, all node / environmental
attributes with "quantum" in their name must be munged to say
"neutron" instead.  Make these changes.

Run chef-client everywhere!


Known Issues (Ubuntu)
---------------------

* In 4.1.3 and 4.2, rabbitmq now binds to the management network.  If
  you are colocating a chef server on the controller, you need to
  ensure that chef server is not using 127.0.0.1 as its rabbitmq
  server address and is instead using the appropriate ip address from
  the management network.

  Something like the following in chef-server.rb should work:
  rabbitmq["node_ip_address"] = "#{node['ipaddress']}"
  rabbitmq["vip"] = "#{node['ipaddress']}

  node['ipaddress'] may not be the right ip, though!

* At present, qemu-kvm does not update cleanly.  You can avoid any
  issues with this by manually upgrading qemu-kvm prior to starting
  the chef-client run.  If you forget to do this, fear not.  The run
  will fail, at which point you can manually upgrade qemu-kvm and then
  run chef-client again with no issues.  Currently this takes a couple
  of runs.  Make sure you've got the havana repos set up.  I've filed
  a bug: https://bugs.launchpad.net/ubuntu/+source/qemu/+bug/1243403

  apt-get install -y qemu-kvm qemu-utils qemu-system-common qemu-system-x86


* The dependencies on the nova-common package in ubuntu are incorrect,
  resulting in the required version of python-cmd2 not being
  installed.  This means you get stack traces on nova-manage service
  list, for example.  This can be resolved by apt-get install
  python-cmd2.  We've filed a package bug here:
  https://bugs.launchpad.net/ubuntu/+source/nova/+bug/1242925

* Horizon does not depend on python-django1.5, but django 1.5 is
  provided in the havana repository.  At the end of the upgrade
  process, django 1.4 will remain installed.  New installations,
  though, will have django 1.5.  This does not seem to cause any
  trouble, but it may not be desirable.  A full apt-get upgrade will
  resolve this issue.  This could be resolved by upgrading the package.

Known Issues (Centos6)
------------

* In 4.1.3 and 4.2, rabbitmq now binds to the management network.  If
  you are colocating a chef server on the controller, you need to
  ensure that chef server is not using 127.0.0.1 as its rabbitmq
  server address and is instead using the appropriate ip address from
  the management network.

  Something like the following in chef-server.rb should work:
  rabbitmq["node_ip_address"] = "#{node['ipaddress']}"
  rabbitmq["vip"] = "#{node['ipaddress']}

  node['ipaddress'] may not be the right ip, though!

* Openstack Dashboard wants to upgrade the package 
  "python-django-openstack-auth.noarch" to version "1.1.2-1". This
  causes the dashboard to fail authentication and die. The specific error is,
  '"POST /auth/login/ HTTP/1.1" 403 1006' and is only seen when in single 
  server mode. To get around this, install the the previous package version 
  "1.0.11-1.el6". Presently you have to downgrade to the "1.0.11-1.el6" 
  package. 
  Bug filed Here: https://bugzilla.redhat.com/show_bug.cgi?id=1000391

