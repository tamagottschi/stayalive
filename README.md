# StayAlive

version 0.1.0<br>
by Jan Gottschick (c) 2016

Monitor and control your own hosted service simply with a client on your own workstation or regularly using a nano service

## Introduction

You create a web application or mobile app for your customer, which requires a related **backend service**, e.g. running inside an application server like _node.js_. Especially, customers with low budget are looking forward to a cost inexpensive solution for the daily operations of their app. Therefor a self service operations management is required.

It must identify problems with your service within a narrow time frame and the customer should be able to fix smaller problems itself, e.g. start/stop the service. The customer should further be able to use its existing hosting environment, e.g. web space hosting with a ssh access, and it should not be required to rent a separate management server.

The figure below shows the basic idea of **StayAlive**. **StayAlive** allows you easily to generate an app, which could be used as a control app on the local workstation of your customer and/or your own support. Further, the app can be deployed in the cloud as a watcher app, which checks regularly if your service is _healthy_.

![alt text](Stayalive.png)

Generating a **StayAlive** app means that **StayAlive** is framework written in the language _Go_. You can write a _StayAlive playbook_, which describes all actions for the healthy checks and operations as well all customer specific configuration values (including the customer secret keys!!). The basic format of a _playbook_ is YAML or JSON. The concrete playbook format is described later in detail. The _StayAlive framework_ translates the _StayAlive playbook_ into go source files and compile it together with the required libraries to your customer-specific _StayAlive App_. The _StayAlive App_ can be packaged within an installer for Mac OSX or Windows.

The customer should install its specific _StayAlive App_ on its personal workstation. After the _StayAlive app_ have been started by the customer the customer can open web browser and start the web application provided by the _StayAlive app_ locally. The _StayAlive app_ is a control app, which provides functions to do a detailed health check of your backend service, and functions to control your service.

![StayAlive App User Interface](StayaliveUI.png)

Additionally the _StayAlive app_ can be installed in the cloud as a nano service, e.g. using Amazon AWS Lambda. The _StayAlive app_ works as a _nano watcher_, which do regularly basic health checks of your backend service. Error conditions will be sent by email to the customer. Such a nano service will be inexpensive and requires no additional resources like an additional hosted server specially for operations management.

## StayAlive playbook

The **StayAlive playbook** describes all the customer solution specific actions and configuration values. The supported basic formats are YAML, JSON, TOML and XML. The following scheme describes the playbook content.

### global section

The global defines general configuration values.

* app: «name of the customer application»
* title: «the title shown in the header»
* version: «semantic version number of the playbook»
* author: «name of the author of the playbook»
* support: «email address for the support of the app»
* password: «password required to access the web application»
* logo: «path to the image file shown as logo in the header»
* email:
	* to: «email address of the customer, to which error message logs will be sent»
	* from: «email address to use as sender of the error message logs»
	* host: «email server used to sent an email»
	* user: «user used to authenticate against the email server for outgoing emails»
	* password: «password used to authenticate against the email server for outgoing emails»

### healthy checks section

The _healthy checks_ will be executed, if the _StayAlive app_ will be started as _nano watcher_, e.g. as a Amazon AWS Lambda function. The healthy checks are a set of check functions, which will be executed and their result evaluated in each run. If an error will be found, the error message log will be sent via email to the _support email address_.

* healthy-checks: «list of check functions»

The _http-get_ check function calls a URL using _HTTP get_ and compares the returned result against the given expectations.

* http-get:
	* behaviour: «text describes the expected behaviour of the check»
	* url: «URL for be request»
	* timeout: maximum response time in milliseconds for a request»
	* result-code: «expected result code defined by a regular expression, which must match»
	* result-body: «expected result body defined by a regular expression, which must match»

The _http-post_ check function calls a URL using _HTTP post_ and the given body. It compares the returned result against the given expectations.

* http-post:
	* behaviour: «text describes the expected behaviour of the check»
	* url: «URL for be request»
	* body: «content of the body sent with the request»
	* timeout: maximum response time in milliseconds for a request»
	* result-code: «expected result code defined by a regular expression, which must match»
	* result-body: «expected result body defined by a regular expression, which must match»

The _ping_ check function calls a URL using a network ping. It compares the returned result against the given expectations.

* ping:
	* behaviour: «text describes the expected behaviour of the check»
	* ip-address: «ip address to be used»
	* timeout: maximum response time in milliseconds for a request»

### detailed checks section

The _detailed checks_ will be executed, if the _StayAlive app_ will be started as _control app_ on the local workstation. The detailed checks are a set of check functions, which will be executed if manually triggered in the web GUI and their result evaluated in each run. If an error will be found, the test result will immediately shown to the user (customer) and a more detailed log is shown, too.

* detailed-checks: «list of check functions»

The check functions are identical with the one from the healthy checks.

### tasks section

You can define your own tasks, which can be executed on demand by the user of the _StayAlive app_, e.g. by pressing a button. The output of the execution is shown in the log area.

* tasks: «set of tasks»

The following task types are available:

* ssh:
	* name: «name of the task»
	* title: «title shown, e.g. on the button»
	* target: «server address, where the tasks will be started»
	* user: «user to use for authentication»
	* key: «private ssh key for authentication»
	* command: «shell command to execute on the target»

### help section

The _StayAlive app_ can show a help text to the user, with the most important information about the usage of the app and perhaps a small description.

* help: «help text using markdown formatting»

### example

This is an example for a playbook used to generate a _StayAlive app and watcher service_.

	app: guestbook
	title: Guestbook for my customer
	version: 0.1.0
	author: Max Mustermann
	support: guestbook-support@my-company.com
	password: secret
	email:
		to: guestbook-error@customer-company.com
		from: guestbook@customer-company.com
		host: mail.customer-company.com
		user: guestbook
		password: moresecret

	healthy-checks:
		- http-get:
			behaviour: Request the ping web service
			url: http://app.customer-company.com/ping
			timeout: 10000
			result-code: 200

	detailed-checks:
		- http-get:
			behaviour: Request the ping web service
			url: http://app.customer-company.com/ping
			timeout: 10000
			result-code: 200
		- ping:
			behaviour: Ping the server hosting the app
			host: app.customer-company.com

	tasks:
		- ssh:
			name: restart
			title-de: Neustart
			target: app.customer-company.com
			user: ssh-12345
			key: |
				-----BEGIN RSA PRIVATE KEY-----
				...
				-----END RSA PRIVATE KEY-----
			command: ~/app/restart.sh

## Generating a StayAlive app

## License

see LICENSE file
