#monitoring_plugins

This plugins work with Nagios and Icinga2.

Documentation focuses on use with Icinga2, becasue I use it.

## check_fleet

Check state of systemd units deployed to CoreOS distributed init system Fleet.

It uses the [Fleet APIv1](https://github.com/coreos/fleet/blob/master/Documentation/api-v1.md) over unix socket or network port.

### Installation

* download ``check_fleet`` and place into ``PluginDir`` which is defined in ``/etc/icinga2/constants.conf``.
* ``chmod +x check_fleet``
* configure commands.conf and services.conf (see examples) 


### Configuration example for Icinga2

in ``/etc/icinga2/conf.d/commands.conf``

    object CheckCommand "my-fleet" {
        import "plugin-check-command"
    
      command = [ PluginDir + "/check_fleet" ]
    
      arguments = {
        "-n" = {
          required = true
          value = "$fleet_name$"
        }
        "-s" = {
          required = true
          value = "$fleet_status$"
        }
        "-c" = "$fleet_cache_ttl$"
        "-h" = "$fleet_hostname$"
        "-p" = "$fleet_port$"
      }
    }



in ``/etc/icinga2/conf.d/services.conf``
    
    apply Service "fleet-unit2-health" {
      # will return error if service exists
      import "generic-service"
    
      check_command = "my-fleet"
    
      vars.fleet_name = "missingUnit.service"
      vars.fleet_state = "missing""
    }
    apply Service "fleet-unit2-health" {
      # will return error if service doesn't exists or is not active 
      import "generic-service"
    
      check_command = "my-fleet"
    
      vars.fleet_name = "service@host.service"
      vars.fleet_state = "!active"
    
      assign where host.name == NodeName
    }
    apply Service "fleet-unit3-health" {
      # will return error if service exists and is not active, missing service is OK 
      import "generic-service"
    
      check_command = "my-fleet"
    
      vars.fleet_name = "service@host.service"
      vars.fleet_state = "active"
    
      assign where host.name == NodeName
    }

## Development

    sudo ssh -nNT -oStrictHostKeyChecking=no -L /var/run/fleet2.sock:/var/run/fleet.sock -i /root/.ssh/coreos.pem core@<SERVER>
    sudo chmod 777 /var/run/fleet2.sock

## Resources

* https://exchange.icinga.org/faq#markdown!
* http://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/monitoring-basics#check-commands
* https://github.com/coreos/fleet/blob/master/Documentation/api-v1.md
