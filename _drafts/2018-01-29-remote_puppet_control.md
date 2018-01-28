---
layout: blog
title: Tip of the Week 57 - Trigger Puppet agent runs from remote
---

Example42's [psick control-repo](https://github.com/example42/psick) has several features which allows user to manage most of the typical infrastructure tasks easily.

Since the release of [Puppet Tasks](https://puppet.com/resources/solution-brief/puppet-tasks) there are [several infrastructure remote commands](https://forge.puppet.com/example42/psick/tasks) available in PSICK:

- psick::system_update - Update all packages on a system
- psick::puppet_unlock - Remove Puppet lockfiles
- psick::puppet_install - Install Puppet agent on a node
- psick::puppet_enable_noop - Enable noop option in Puppet agent config
- psick::puppet_agent - Run Puppet agent on a node

This posting will dig into how to trigger a Puppet agent run from remote.

Generally there are two possible solutions on Puppet Enterprise:

1. use Puppet tasks
2. use PCP broker and a PXP agent module

The solution with Puppet Tasks is working, but has the downside of not directly
showing the result from puppet agent run, while the run takes place.

The solution with PCP broker and a PXP agent module is not working, as we can not
 access the PCP broker api (it is a private API considered to be used by Puppet
Enterprise only).

## Tasks via Orchestrator (and PXP)

Access to [Orchestrator API](https://puppet.com/docs/pe/2017.3/orchestrator/orchestrator_api_v1_endpoints.html) requires a [token](https://puppet.com/docs/pe/2017.3/rbac/rbac_token_auth_intro.html) and proper [RBAC permissions](https://puppet.com/docs/pe/2017.3/rbac/managing_access.html).

The generated token can then be used in a https call to authorize at api using headers.
Task parameters are added to the payload of the request. Affected nodes can be listed as arry.

The following exmaple shows how to build the `curl` command call:

    curl -k -X POST \
      -H "Content-Type: application/json" \
      -H 'X-Authentication:<token>' \
      https://<mom or compile master>:8143/orchestrator/v1/command/task \
      -d '{
        "environment" : "production",
        "task" : "psick::puppet_agent",
        "params" : {
          "noop" : true,
          "puppet_master" : "<compile master to use for this specific agent run>"
        },
        "scope" : {
          "nodes" : ["<node1>", "<node2>"]
        }
      }'

This will return tbe following Output:

    {
      "job" : {
        "id" : "https://<mom or comile master>:8143/orchestrator/v1/jobs/12",
        "name" : "12"
      }
    }

Read result from job:

    curl -k -X GET \
      -H 'X-Authentication:<token>' \
      https://<mom or comile master>:8143/orchestrator/v1/jobs/12

End of output while still running:

      "node_count" : 1,
      "node_states" : {
        "running" : 1
      }

End of output when finished:

      "node_count" : 1,
      "node_states" : {
        "finished" : 1
      }

Read node results:

    curl -k -X GET \
      -H 'X-Authentication:<token>' \
      https://<mom or compile master>:8143/orchestrator/v1/jobs/12/nodes

Output:

    {
      "items" : [ {
        "finish_timestamp" : "2018-01-24T14:41:14Z",
        "transaction_uuid" : null,
        "start_timestamp" : "2018-01-24T14:40:18Z",
        "name" : "<node1>",
        "duration" : 55.795,
        "state" : "finished",
        "details" : { },
        "result" : {
                  "_output" : "\u001B[0;32mInfo: Using configured environment 'production'\u001B[0m\n\u001B[0;32mInfo: Retrieving pluginfacts\u001B[0m\n\u001B[0;32mInfo: Retrieving plugin\u001B[0m\n\u001B[0;32mInfo: Loading facts\u001B[0m\n\u001B[0;32mInfo: Caching catalog for <node1>\u001B[0m\n\u001B[0;32mInfo: Applying configuration version '7a4be91 - Alessandro Franceschi, Sun Jan 14 18:15:44 2018 +0100 : run acceptance tests also for newer puppet versions (#220) (#221)'\u001B[0m\n\u001B[mNotice: /Stage[main]/Psick::Dns::Resolver/File[/etc/resolv.conf]/content: \n--- /etc/resolv.conf\t2018-01-24 14:31:51.428194104 +0000\n+++ /tmp/puppet-file20180124-20270-1wxros6\t2018-01-24 14:40:44.602327849 +0000\n@@ -1,3 +1,3 @@\n-# Generated by NetworkManager\n-search pe.psick.io\n-nameserver 10.0.2.3\n+#File managed by Puppet\n+nameserver 8.8.8.8\n+nameserver 8.8.4.4\n\u001B[0m\n\u001B[0;32mInfo: Computing checksum on file /etc/resolv.conf\u001B[0m\n\u001B[0;32mInfo: FileBucket got a duplicate file {md5}b9dfc6d9764870be83fe35ecf2cfc5f3\u001B[0m\n\u001B[0;32mInfo: /Stage[main]/Psick::Dns::Resolver/File[/etc/resolv.conf]: Filebucketed /etc/resolv.conf to puppet with sum b9dfc6d9764870be83fe35ecf2cfc5f3\u001B[0m\n\u001B[mNotice: /Stage[main]/Psick::Dns::Resolver/File[/etc/resolv.conf]/content: \n\u001B[0m\n\u001B[mNotice: /Stage[main]/Psick::Dns::Resolver/File[/etc/resolv.conf]/content: content changed '{md5}b9dfc6d9764870be83fe35ecf2cfc5f3' to '{md5}3ccdb679ea166bdf52104b3ae3a4499d'\u001B[0m\n\u001B[mNotice: Applied catalog in 29.16 seconds\u001B[0m\n"
        },
        "latest-event-id" : 41,
        "timestamp" : "2018-01-24T14:41:14Z"
      } ],
      "next-events" : {
        "id" : "https://<node1>:8143/orchestrator/v1/jobs/14/events?start=42",
        "event" : "42"
      }
    }


## Using PCP Broker and a PXP Module:

ATTENTION: this solution is NOT working at the moment!

First we need a pxp-agent module on the nodes:

For development purpose the module can be executed locally:

    sudo echo "{\"input\":{\"flags\":[\"--noop\", \"--server=puppet.pe.psick.io\"]}, \"configuration\" : {\"puppet_bin\" : \"/opt/puppetlabs/bin/puppet\"}}" | /etc/puppetlabs/pxp-agent/modules/puppet_agent run

Not working, as the API is unknown. Could be something simlar to the following:

    curl -k -X POST \
      -H "Content-Type: application/json" \
      -H 'X-Authentication:0MDuxrsDQnQbzGn1P_4sWu4hgzg8AvQk0sebRuReGiZI' \
      https://puppet.pe.psick.io:8142/orchestrator/v1/command/task \
      -d '{
        "properties" : {
          "notify_outcome" : true,
          "module" : "puppet_agent",
          "action" : "run",
          "params" : {
            "flags" : [ "--noop", "--server=puppet.pe.psick.io" ]
          }
        },
        "required" : ["transaction_id", "notify_outcome", module", "action"],
        "additionalProperties" : false
      }'


Happy hacking,

Martin Alfke
