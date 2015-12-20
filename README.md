## check_fleet

### Icinga2 configuration example

in /etc/icinga2/conf.d/commands.conf

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



in /etc/icinga2/conf.d/services.conf
    
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


## Resources

* https://help.github.com/articles/github-flavored-markdown/
* http://docs.icinga.org/icinga2/latest/doc/module/icinga2/toc#!/icinga2/latest/doc/module/icinga2/chapter/monitoring-basics#check-commands
