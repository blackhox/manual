# Setting up Validator Monitoring for Cosmos #
This is a detailed guide on how to set up validator monitoring for Cosmos-based blockchains using Prometheus and Grafana.
This guide is for validators who need to set up basic monitoring of their nodes. This manual does not cover all possible monitoring options. 
To set up deeper monitoring, it is recommended to refer to the official documentation.
# Set up a Prometheus server #
First, we need to set up the Prometheus server. 
It will collect all the node monitoring parameters. Prometheus is a database that feeds various diagnostic parameters in near real time.
To install the Prometheus server, it is strongly recommended to allocate a separate server, of course, you can run it along with other programs, but the performance of such a system may be impaired. It is strongly not recommended to run Prometheus on the same server as a sentry or validator, as this will lead to unnecessary load on the server, and the lack of system resources in turn will lead to block skipping.
First of all, let's start the installation by updating the OS and installing security software: **fail2ban**
It won't solve every possible security problem, but it's better than doing nothing at all.
