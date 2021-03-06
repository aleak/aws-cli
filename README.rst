=======
aws-cli
=======

.. image:: https://travis-ci.org/aws/aws-cli.png?branch=develop
   :target: https://travis-ci.org/aws/aws-cli
   :alt: Build Status

This package provides a unified command line interface to many
Amazon Web Services.

The currently supported services include:

* AWS CloudFormation
* AWS Data Pipeline
* AWS Direct Connect
* AWS Elastic Beanstalk
* AWS Identity and Access Management
* AWS Import/Export
* AWS OpsWorks
* AWS Security Token Service
* AWS Storage Gateway
* Amazon CloudWatch
* Amazon Elastic Compute Cloud
* Amazon Elastic MapReduce
* Amazon Elastic Transcoder
* Amazon Redshift
* Amazon Relational Database Service (Beta)
* Amazon Simple Email Service
* Amazon Simple Notification Service
* Amazon Simple Queue Service
* Amazon Simple Storage Service
* Amazon Simple Workflow Service
* Auto Scaling
* Elastic Load Balancing


The aws-cli package should work on Python versions 2.6.x - 3.3.x.

------------
Installation
------------

The easiest way to install aws-cli is to use ``easy_install`` or ``pip``::

    $ easy_install awscli

or, if you are not installing in a ``virtualenv``::

    $ sudo easy_install awscli

Using ``pip``, it would simply be::

    $ pip install awscli

or::

    $ sudo pip install awscli

This will install the aws-cli package as well as all dependencies.  You can
also just clone the git repo or download the tarball.  Once you have the
awscli directory structure on your workstation, you can just::

    $ cd <path_to_awscli>
    $ python setup.py install

------------------
Command Completion
------------------

The aws-cli package includes a very useful command completion feature.
This feature is not automatically installed so you need to configure it manually.
To enable tab completion for bash use the built-in command ``complete``::

    $ complete -C aws_completer aws

For tcsh::

    $ complete aws 'p/*/`aws_completer`/'

You should add this to your startup scripts to enable it for future sessions.

For zsh please refer to bin/aws_zsh_completer.sh.  Source that file::

    $ source bin/aws_zsh_completer.sh

For now the bash compatibility auto completion (bashcompinit) is used.
For further details please refer to the top of bin/aws_zsh_completer.sh.

---------------
Getting Started
---------------

Before using aws-cli, you need to tell it about your AWS credentials.  You
can do this in several ways:

* Environment variables
* Config file
* IAM Role

To use environment variables, do the following::

    $ export AWS_ACCESS_KEY_ID=<access_key>
    $ export AWS_SECRET_ACCESS_KEY=<secret_key>

To use a config file, create a configuration file like this::

    [default]
    aws_access_key_id=<default access key>
    aws_secret_access_key=<default secret key>
    region=us-west-1  # optional, to define default region for this profile

    [profile testing]
    aws_access_key_id=<testing access key>
    aws_secret_access_key=<testing secret key>
    region=us-west-2

As you can see, you can have multiple ``profiles`` defined in this
configuration file and specify which profile to use by using the
``--profile`` option.  If no profile is specified the ``default``
profile is used.  Once you have created the config file, you need to
tell aws-cli where to find it.  Do this by setting the appropriate
environment variable::

    $ export AWS_CONFIG_FILE=/path/to/config_file

The final option for credentials is highly recommended if you are
using aws-cli on an EC2 instance.  IAM Roles are
a great way to have credentials installed automatically on your
instance.  If you are using IAM Roles, aws-cli will find them and use
them automatically.

----------------------------
Other Configurable Variables
----------------------------

In addition to credentials, a number of other variables can be
configured either with environment variables, configuration file
entries or both.  The following table documents these.

=========== ============ ===================== ============================
Variable    Config Entry Environment Variable  Description
=========== ============ ===================== ============================
profile                  AWS_DEFAULT_PROFILE   Default profile name
----------- ------------ --------------------- ----------------------------
region      region       AWS_DEFAULT_REGION    Default AWS Region
----------- ------------ --------------------- ----------------------------
config_file              AWS_CONFIG_FILE       Alternate location of config
----------- ------------ --------------------- ----------------------------
output      output       AWS_DEFAULT_OUTPUT    Default output style
----------- ------------ --------------------- ----------------------------
access_key  access_key   AWS_ACCESS_KEY_ID     AWS Access Key
----------- ------------ --------------------- ----------------------------
secret_key  secret_key   AWS_SECRET_ACCESS_KEY AWS Secret Key
----------- ------------ --------------------- ----------------------------
token       token        AWS_SECURITY_TOKEN    AWS Token (temp credentials)
=========== ============ ===================== ============================

^^^^^^^^
Examples
^^^^^^^^

If you get tired of specifying a ``--region`` option on the command line
all of the time, you can specify a default region to use whenever no
explicit ``--region`` option is included using the ``region`` variable.
To specify this using an environment variable::

    $ export AWS_DEFAULT_REGION=us-west-2

To include it in your config file::

    [default]
    aws_access_key_id=<default access key>
    aws_secret_access_key=<default secret key>
    region=us-west-1  # This will be used as the defaul

Similarly, the ``profile`` variable can be used to specify which profile to use
if one is not explicitly specified on the command line via the
``--profile`` option.  To set this via environment variable::

    $ export AWS_DEFAULT_PROFILE=testing

The ``profile`` variable can not be specified in the configuration file
since it would have to be associated with a profile and would defeat the
purpose.

----------------------------------------
Accessing Services With Global Endpoints
----------------------------------------

Some services, such as AWS Identity and Access Management (IAM),
AWS Security Token Service (STS), and Amazon Simple Email Service (SES)
have a single, global endpoint rather than different endpoints for
each region.

To make access to these services simpler, aws-cli will automatically
use the global endpoint unless you explicitly supply a region (using
the ``--region`` option) or a profile (using the ``--profile`` option).
Therefore, the following::

    $ aws iam list-users

Will automatically use the global endpoint for the IAM service
regardless of the value of the ``AWS_DEFAULT_REGION`` environment
variable or the ``region`` variable specified in your profile.

-----------------------
Complex Parameter Input
-----------------------

Many options that need to be provided are simple string or numeric
values.  However, some operations require complex data structures
as input parameters.  These options must be provided as JSON data
structures, either on the command line or in files.

For example, consider the command to authorize access to an EC2
security group.  In this case, we will add ingress access to port 22
for all IP addresses::

    $ aws ec2 authorize-security-group-ingress --group-name MySecurityGroup --ip-permissions '{"from_port":22,"to_port":22,"ip_protocol":"tcp","ip_ranges":["0.0.0.0/0"]}'

You could also place the JSON in a file, called port22.json for example,
and use this::

    $ aws ec2 authorize-security-group-ingress --group-name MySecurityGroup --ip-permissions /path/to/port22.json

--------------------------
File-based Parameter Input
--------------------------

Some parameter values are so large or so complex that it would be easier
to place the parameter value in a file and refer to that file rather than
entering the value directly on the command line.

Let's use the ``authorize-security-group-ingress`` command shown above.
Rather than provide the value of the ``--ip-permissions`` parameter directly
in the command, you could first store the values in a file.  Let's call
the file ip_perms.json::

    {"from_port":22,
     "to_port":22,
     "ip_protocol":"tcp",
     "ip_ranges":["0.0.0.0/0"]}

Then, we could make the same call as above like this::

    aws ec2 authorize-security-group-ingress --group-name MySecurityGroup --ip-permissions file:ip_perms.json

The ``file:`` prefix on the parameter value signals that the parameter value
is actually a reference to a file that contains the actual parameter value.
aws-cli will open the file, read the value and pass use that value as the
parameter value.

This is also useful when the parameter is really referring to file-based
data.  For example, the ``--user-data`` option of the ``aws ec2 run-instances``
command or the ``--public-key-material`` parameter of the
``aws ec2 import-key-pair`` command.

-------------------------
URI-based Parameter Input
-------------------------

Similar to the file-based input described above, aws-cli also includes a
way to use data from a URI as the value of a parameter.  The idea is exactly
the same except the prefix used is ``https:`` or ``http:``::

    aws ec2 authorize-security-group-ingress --group-name MySecurityGroup --ip-permissions http://mybucket.s3.amazonaws.com/ip_perms.json

--------------
Command Output
--------------

The default output for commands is currently JSON.  This may change in the
future but for now it provides the most complete output.  You may find the
[jq](http://stedolan.github.com/jq/) tool useful in processing the JSON
output for other uses.

There is also an ASCII table format available.  You can select this
style with the ``--output`` option or you can make this style your default
output style via environment variable or config file entry as described above.
