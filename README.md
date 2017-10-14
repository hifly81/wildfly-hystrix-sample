# Wildfly Hystrix and Turbine Example

__This example is based on the work of<br>
https://github.com/siamaksade/wildfly-swarm-hystrix-example <br>
Thanks for the big work!__

This is the version of the original example for Wildfly/EAP without the usage of Wildfly Swarm.

Wildfly Example using Hystrix as the circuit-breaker. This example is composed of:
* __Employee REST Service:__ JAX-RS service that returns the list of employees. This service randomly generates error and timed-out responses.
* __Payroll REST Service:__ JAX-RS service that invokes Employee service using Hystrix
* __Turbine:__ Netflix components for aggregating streams of json data
* __Hystrix Dashboard:__ a dashboard for visualizing aggregated data streams


# Run on OpenShift

Employee Service, Payroll Service and Turbine will be created as new app in OCP using the eap7 image

Hystrix Dashboard will be created as new app in OCP using a docker image from docker hub: mlabouardy/hystrix-dashboard:latest

These are the commands to run on OCP to create the entire project:

	$ oc new-project hystrix-wildfly
	$ oc policy add-role-to-user cluster-reader system:serviceaccount:hystrix-wildfly:default

	$ oc new-app eap70-openshift~https://github.com/hifly81/wildfly-hystrix-sample --context-dir=employee-service --name=employee-app
    $ oc new-app eap70-openshift~https://github.com/hifly81/wildfly-hystrix-sample --context-dir=payroll-service --name=payroll-app -l hystrix.enabled='true'
    $ oc new-app eap70-openshift~https://github.com/hifly81/wildfly-hystrix-sample --context-dir=turbine --name=turbine
    $ oc new-app mlabouardy/hystrix-dashboard:latest --name=hystrix-dashboard

    $ oc expose service employee-app
    $ oc expose service payroll-app
    $ oc expose service turbine
    $ oc expose service hystrix-dashboard


Employee service and Payroll service will exposed at:<br>
http://employee-app-hystrix-wildfly.127.0.0.1.nip.io/employees<br>
http://payroll-app-hystrix-wildfly.127.0.0.1.nip.io/payroll

If you get an "Internal Server Error" for some of the Employee requests, it's by design! The service simulates a certain ration of errors and timeouts.

# Hystrix Dashboard

The dashboard will be available at<br>
http://hystrix-dashboard-hystrix-wildfly.127.0.0.1.nip.io/hystrix

Hystrix Dashboard must be configured to add the stream from Turbine; the url of the stream is:<br>
http://<service_internal_ip>:8080/turbine-1.0.0-SNAPSHOT/turbine.stream

Generate some load on the Payroll service and monitor the endpoints through Hystrix Dashboard:

	$ ab -n 100 http://payroll-app-hystrix-wildfly.127.0.0.1.nip.io/payroll