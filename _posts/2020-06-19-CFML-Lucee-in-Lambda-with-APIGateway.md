---
layout: post
title:  "Running Coldfusion/CFML in AWS Lambda"
date:   2020-06-19 19:18:01 +1000
categories: aws lambda apigateway cfml lucee
---

In a previous life I had worked on some Coldfusion projects, it was an interesting language if you are not familiar you can read about the language itself [here](https://en.wikipedia.org/wiki/ColdFusion_Markup_Language).  ColdFusion is owened, built and supported by Adobe, though along the way a few open source CFML engines, Lucee, OpenDB and many years ago one or 2 more half implemented engines.

Interestingly in 2009 OpenBD was is a state where it was deployable to the cloud via Google App engine, it's hard to believe this was 11 years ago but here is a video I took on My 2008 Mac Book.  I don’t remember much about it but there was no file system and there was a Virtual File System what used DataStore.
https://youtu.be/BFJF_0BGIS8

<iframe width="600" height="400" src="https://www.youtube.com/embed/BFJF_0BGIS8" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

Nostalgia aside fast-forward 11 years and [Lucee](https://lucee.org/) by far is the most dominant Open Source implementation and has the ability to run CFML in AWS Lambda leveraging [Fuseless](https://fuseless.org/).  

![Function](/assets/post/2020-06-19-CFML-Lucee-in-Lambda-with-APIGateway/fuseless.png "Function")

Up until a few weeks ago this was broken with any recent version of [SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html) but my buddy Malcom Crum and I poked around and saw the changes in the java code for the java runtime in lambda and made the appropriate fixes.  At the time I compiled fuseless locally and if there is any interest I can do a second post on how to step through that.  We also updated the init scripts o pull down a more recent version of lucee.

I have built a sample repo with all the jars included ( [https://github.com/kukielp/cfml-samples/tree/master/cfml-on-lambda](https://github.com/kukielp/cfml-samples/tree/master/cfml-on-lambda) )

Satisfying the requirements of fuseless you "should" be able to clone that repo, build the project and use SAM to push the project and expose an api.  You'll of course have to setup a database and change the Security Groups and VPC.

The example program is running as:  [https://8wgei5rj1f.execute-api.ap-southeast-2.amazonaws.com/Prod/demo.cfm](https://8wgei5rj1f.execute-api.ap-southeast-2.amazonaws.com/Prod/demo.cfm)

The demo includes:
- Running cfml in lambda
- Demonstrating the applicaiton scope persistence between invocations
- Configuring a DSN ( Data source ) in Application.cfc
- Lifting ENV variables and leveraging those in CMFL code
- Querying a database
- Setting CORS ( or any other ) headers for cross site access
- Object/cfc creation
- Serialisation of cfml object to JSON
- Demostrating on onError and onMissingTemplate in application.cfc

Of course, there is some cold start, Lucee is a jee application so that’s not really avoidable though there are ways to keep lambdas warm as well as provisioned concurrency to keep some lambdas always hot.

As I haven’t used CFML for years the only new feature I took advantage of was environment variables where I pass those in from SSM ( you can see this in template.yaml )


{% highlight javascript %}
component {

	this.name="cfmlServerless";
	this.applicationTimeout = CreateTimeSpan(10, 0, 0, 0); //10 days
	this.sessionManagement=false;
	this.clientManagement=false;
	this.setClientCookies=false;
	
	public function onRequest(string path) {
		setting enablecfoutputonly="true" requesttimeout="180" showdebugoutput="true";
		application.counter++;
		include listLast(arguments.path,'/');
	}

	this.datasources["pgjdbc"] = {
		class: 'org.postgresql.Driver'
		, connectionString: 'jdbc:postgresql://' & server.system.environment.DB_CONNECTION_STRING
		, username: server.system.environment.DB_USERNAME
		, password: server.system.environment.DB_PASSWORD
	}

	this.defaultdatasource = "pgjdbc"

	function onApplicationStart() {
		application.counter = 0;
		application.resultsArray = [];
		return true;
	}

	function getCounter() {
		return application.counter;
	}

	public function getLambdaContext() {
		//see https://docs.aws.amazon.com/lambda/latest/dg/java-context-object.html
		return getPageContext().getRequest().getAttribute("lambdaContext");
	}

	public void function logger(string msg) {
		getLambdaContext().getLogger().log(arguments.msg);
	}

	public string function getRequestID() {
		if (isNull(getLambdaContext())) {
			//not running in lambda
			if (!request.keyExists("_request_id")) {
				request._request_id = createUUID();
			}
			return request._request_id;
		} else {
			return getLambdaContext().getAwsRequestId();
		}
	}

	function onMissingTemplate(){
		include '404.cfm';
	}

	function onError( any Exception, string EventName ) {
		writeOutput("Some error has occured");
		abort;
	}
}
{% endhighlight %}

Setting the headers for CORS was simple in cfml.  While this demo runs on the same page while testing the html, I was running the HTML page locally calling the remote endpoints on lambda.

{% highlight xml %}
<cfheader name="Access-Control-Allow-Origin" value="*" />
<cfcontent type="text/html; charset=utf-8">
<cfprocessingdirective pageEncoding="utf-8">

<cfquery name="q" datasource="pgjdbc">
    select      customer_id, contact_title
    from        customers
    <cfif structKeyExists(url, 'filter')>
        where       contact_title like <cfqueryparam value="%#filter#%" cfsqltype="CF_SQL_VARCHAR" />
    </cfif>
</cfquery>
<cfdump var="#q#" />
{% endhighlight %}

You might also ask "How can I run this locally?” well you can with SAM.  Open 2 terminals, in terminal one run:

```bash 
gradle build -t
```

In terminal 2 run:
```bash
sam local start-api
```

You can use the browser to see the result, the browser will always request a favicon.ico invoking a second call, so I tend to use Postman.  Running this locally will always incur cold start on each request.

TO deploy the app run:
In terminal 2 run:
```bash
sam deploy --guided
```
Note that you will have to specify Secruity groups and a VPC id or simply remove those lines if you do not need to run in a VPC ro do not have a database.


I'm not sure who might be using cfml in lambda or who is interested in a deeper dive, if I have some interest I can do a second or thrird post on this subject.  Hope you enjoy.