# awsenv

Python utility to manage AWS environment variables for multiple profiles,
especially with cross-account access enabled.

Specifically, `awsenv`:

 - Determines a profile to use based on CLI input or the `AWS_PROFILE` environment variables
 - Loads AWS configuration from `~/.aws/config` (or `AWS_CONFIG_FILE`) and `~/.aws/credentials`
 - Invokes STS Assume Role if the profile defines a `role_arn`
 - Prints shell statements to (un)set appropriate environment variables

[![Build Status](https://travis-ci.org/locationlabs/awsenv.png)](https://travis-ci.org/locationlabs/awsenv)


## Credits

`awsenv` is largely based on [mlrobinson]()'s `assume_role.py` script.

 [mlrobinson]: https://gist.github.com/mlrobinson/944fd0e2ad4926ba71c9


## Installation

Use pip:

    pip install awsenv


For `virtualenvwrapper` users, the following is recommended:

    # create a new virtualenv
    mkvirtual awsenv
    
    # install with pip
    pip install awsenv

    # soft link the console script (assumes that ~/bin is in your $PATH)
    ln -s ~/.virtualenvs/awsenv/bin/awsenv ~/bin


## Command Line Usage

Invoke `awsenv` with a profile name (or no arguments for the default profile):

    awsenv myprofile

Or set the `AWS_PROFILE` environment variable and run `awsenv` without arguments:

    export AWS_PROFILE=myprofile
    awsenv

The output will be a series of `export` (or `unset`) statements:

    export AWS_DEFAULT_REGION=us-west-2
    export AWS_SESSION_TOKEN=<redacted>
    export AWS_PROFILE=myprofile
    export AWS_SESSION_NAME=<redacted>
    export AWS_SECRET_ACCESS_KEY=<redacted>
    export AWS_ACCESS_KEY_ID=<redacted>

Evaluate the output of to load it into your shell:

    eval "$(awsenv myprofile)"


## Programmatic Usage

Python programs and scripts that use `botocore` and need cross-account access can use the
underlying library directly to obtain an `AWSProfile` instance. The instance, in turn,
defines its own `create_client` function, which will properly set the profile's region,
access key, secret key, and session token:

    from awsenv.main import get_profile

    profile = get_profile()
    client = profile.create_client("ec2")
    print client.describe_instances()


## Session Caching

`awsenv` will check its current environment for the `AWS_SESSION_NAME`, `AWS_SESSION_TOKEN`,
and `AWS_PROFILE` variables; if these are defined and have a non-expired session, the existing
session will be re-used.
