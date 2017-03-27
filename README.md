awslogs-setup
=========

Set up the AWS CloudWatch Logs Agent from scratch.

Unlike similar roles, this does not call the `awslogs-agent-setup.py` script.
Instead, that script has been decomposed into a set of files, templates and
ansible tasks which can be run to achieve the same (non-interactive) results.

This allows us to clearly separate the agent setup from the agent
configuration.  The latter will be very specific to your system and should
probably be handled by a separate role that has `awslogs-setup` as a dependency
(see below).

A copy of the original `awslogs-agent-setup.py script` is provided in the `/src`
directory for reference.

Requirements
------------

For use with EC2 Linux instances running on AWS.

Role Variables
--------------

You will probably need to set the AWS region that will be used by the
installed `aws-cli`. This defaults to:

    awslogs_setup_region: ap-southeast-2

The setup installs an agent daemon "nanny" script that tries to ensure that
the agent is always running.  This requires a couple of paths to system
commands to be specified. The following defaults should work on most modern
systems:

    awslogs_setup_nanny_ps:      /usr/bin/ps
    awslogs_setup_nanny_cat:     /usr/bin/cat
    awslogs_setup_nanny_grep:    /usr/bin/grep
    awslogs_setup_nanny_service: /usr/sbin/service

If you have quirky (or old) system, however, these can be overridden.

Example Playbook
----------------

This role only sets up the awslogs agent. The agent configuration is left up
to you. This is very simple, but can be very specific to your system.

    - hosts: servers
      vars:
        awslogs_setup_region: us-east-1   # or set via inventory

      roles:
        - tartansandal.awslogs-setup

      handlers:
        - name: restart awslogs
          service: awslogs state=restarted

      post_tasks:
        - name: configure awslogs for system log files
          template:
            src: awslogs.conf.j2
            dest: /var/awslogs/etc/awslogs.conf
          notify: restart awslogs

        # use the /var/awslogs/etc/config/ directory for service specific configs
        - name: configure awslogs for nginx log files
          template:
            src: nginx-awslogs.conf.j2
            dest: /var/awslogs/etc/config/nginx.conf
          notify: restart awslogs

The `post_tasks` shown above could be factored out into separate roles that have
`awslogs-setup` as a dependency.

See http://docs.aws.amazon.com/AmazonCloudWatch/latest/DeveloperGuide/AgentReference.html
for details on configuring the agent

ToDo
----

1. Create a systemd configuration to manage the launcher script.

License
-------

BSD

