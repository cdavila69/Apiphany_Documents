![APIphany Logo Blue](http://i.imgur.com/fG9dSvD.png)

# Technical Note 2: Policy Definitions #

## Summary: ##

This document describes policies, policy definitions, policy statements, and policy scopes, and explains how to use the Policy Editor to set a policy at a desired scope. [Global, Product, API and Operations]

## Policy Editor ##

**Description:** 
By default, the Apiphany proxy receives all requests to the APIs managed by Apiphany. It forwards them to the target web services, collects web service reponses, and returns them to the respective API consumers. API publishers can customize this default processing flow by creating policy configurations and applying them to a desired scope (i.e. an API product, API, or an API operation).

A policy configuration affects all requests sent to the selected scope and/or corresponding responses. It also defines which policies should be applied to the requests and responses, and in what order. Individual policies encapsulate functionality that is invoked in the context of a request or a response.

Policies implement API protection, mediation, or transformation actions. Apiphany provides built-in policies as well as the ability to implement custom policies. Each built-in policy is described in a separate technical note.

Policy configurations exist in the form of an XML fragments. Individual policies are represented by XML elements known as policy statements.

Policy statements MUST be enclosed within either <inbound> or <outbound> elements.- <base /> elements represent policy-in-effect inherited from the outer scopes.
         - <inbound> element contains policies to be applied in the inbound direction (from caller to web service).
         - <outbound> element contains policies to be applied in the outbound direction (from web service to caller).
         - Policies are applied in the order they appear.

To **ADD** a policy, position cursor in the policy document to specify the insertion point and click on the associated button for a desired policy statement button.
To **REMOVE** a policy, delete the corresponding policy statement from the policy document.
To **RE-ORDER** a policy, select the corresponding policy statement and cut-and-paste it into a new location within the policy document.


 >	
  	<policies>
		<inbound>
>
		</inbound>
		<outbound>
>
		</outbound>
	</policies>
 >

**Policy Statement:** 

	< policies >, < inbound >, < outbound >

**Example:**
 	
> 		<policies>
    		<inbound>
        		<cross-domain />
    		</inbound>
    		<outbound>
        		<find-and-replace from="Partly Cloudy" to="Sunny" />
    		</outbound>
		</policies>
>

**Where it can be applied:** 
Policy editor, is the default window for policy implementation

**When it should be applied:** 
It always applies, as it is the only window you can use to define policies

**Why it should be applied, why not:** 
Always applies, when needing to apply a policy

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>policies</td>
  <td>Policy statements MUST be enclosed within either <inbound> or <outbound> elements</td>
</tr>
<tr>
  <td>inbound</td>
  <td>element contains policies to be applied in the inbound direction (from caller to web service)</td>
</tr>
<tr>
  <td>outbound</td>
  <td>element contains policies to be applied in the outbound direction (from web service to caller)</td>
</tr>
</table>

## Policy: Get from Cache ##

**Description:**
Perform cache look up and return a valid cached response when available. Appropriately respond to cache validation requests from callers. Use anywhere in the inbound section at API or Operation scopes.

**Policy Statement:** 

	<cache-lookup vary-by-developer=*"true | false"* vary-by-developer-groups=*"true | false"* downstream-caching-type=*"none | private | public"*>
    <vary-by-header>Accept</vary-by-header> <!-- should be present in most cases -->
    <vary-by-header>Accept-Charset</vary-by-header> <!-- should be present in most cases -->
    <vary-by-header>header name</vary-by-header> <!-- optional, can repeated several times -->
   	<vary-by-query-parameter>parameter name</vary-by-query-parameter> <!-- optional, can repeated several times -->
    </cache-lookup>

**Example:**

>		<policies>
>      		<inbound>
>      			<base />
	    		<cache-lookup vary-by-developer="false" vary-by-developer-groups="false" downstream-caching-type="none">
      			<vary-by-query-parameter>version</vary-by-query-parameter>
      			<vary-by-header>Accept</vary-by-header>
      			<vary-by-header>Accept-Charset</vary-by-header>
    			</cache-lookup>		 	
			</inbound>
 			<outbound>
>				<cache-store duration="seconds" caching-mode="cache-on | do-not-cache" />
	     		<base /> 		
 			</outbound>
>		</policies>
>


**Where it can be applied:**
Use in the inbound section at API or operation scopes.

**When it should be applied:**
To reduce the latency to web servers / back-end systems

**Why it should be applied, why not:**
Web proxy servers employ web caches to store previous responses from web servers. Cache store reduces the amount of information that needs to be transmitted across the network, as information previously stored in the cache can often be re-used. This reduces bandwidth and processing requirements of the web server, and reduces latency to users and developers requesting an API; based on the geographic locations of the user, the origin of the API and content delivery server.
	• Cache on – Apiphany caches responses and manages cache-related headers
	• Do not cache – Apiphany does not cache responses and manages cache-related headers
	• Cache neutral – Apiphany neither caches responses nor manages cache-related headers. Equivalent to not having a caching policy configured.


<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>vary-by-developer="true | false"</td>
  <td>*false* by default. Set to *True* to turn cache on per developer key.</td>
</tr>
<tr>
  <td>vary-by-developer-groups="true | false"</td>
  <td>*false* by default. Set to True to turn cache on for all user roles</td>
</tr>
<tr>
  <td>downstream-caching-type="none | private | public"</td>
  <td>*none* set by default; downstream caching is not allowed | *Private* downstream caching it instructs proxies in the path not to cache the response. But it permits browsers to cache the response. | Public downstream caching tells the browser and proxies in the path that the response may be cached. This is good for non-sensitive responses, as caching improves performance.</td>
</tr>
<tr>
  <td>vary-by-header: "Accept"</td>
  <td>Content-Types that are acceptable for the response, Accept: text/plain</td>
</tr>
<tr>
  <td>vary-by-header: Accept-Charset"</td>
  <td>Character sets that are acceptable ,  Accept-Charset: utf-8</td>
</tr>
<tr>
  <td>vary-by-header: "header name"</td>
  <td>request-header examples:	| Accept | Accept-Charset | Accept-Encoding | Accept-Language | Authorization | Expect | From | Host | If-Match  </td>
</tr>
<tr>
  <td>vary-by-query-parameter: "parameter name"</td>
  <td>Enter a single parameter or more than one. Each path parameter is separated by a semicolon. Path parameter names are used to compute cache keys. If none specified, complete URLs are used as keys</td>
</tr>
</tbody>
</table>

## Policy: Store to Cache ##

**Description:**
Cache response according to the specified cache configuration.

**Policy Statement:** 

	cache-store duration="seconds" caching-mode="cache-on | do-not-cache" />

**Example:**

>		<policies>
>			<inbound>
         		<base />
>			      <cache-lookup vary-by-developer=*"true | false"* vary-by-developer-groups=*"true | false"* downstream-caching-type=*"none | private | public"*>
    			<vary-by-header>Accept</vary-by-header> <!-- should be present in most cases -->
    			<vary-by-header>Accept-Charset</vary-by-header> <!-- should be present in most cases -->
    			<vary-by-header>header name</vary-by-header> <!-- optional, can repeated several times -->
   				<vary-by-query-parameter>parameter name</vary-by-query-parameter> <!-- optional, can repeated several times -->
    			</cache-lookup>
        	</inbound>
	 		<outbound>
	     	<base />
				<cache-store duration="3600" caching-mode="cache-on" />
    		</outbound>
>		</policies>

**Where it can be applied:**
Use anywhere in the outbound section at operations scope only.

**When it should be applied:**
To reduce the latency to web servers / back-end systems

**Why it should be applied, why not:**
 Web proxy servers employ web caches to store previous responses from web servers. Cache store reduces the amount of information that needs to be transmitted across the network, as information previously stored in the cache can often be re-used. This reduces bandwidth and processing requirements of the web server, and reduces latency to users and developers requesting an API; based on the geographic locations of the user, the origin of the API and content delivery server.

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>seconds</td>
  <td>Duration (in seconds, default=3600) for the cached entry</td>
</tr>
<tr>
  <td>cache-on | do-not-cache</td>
  <td>Apiphany caches responses and manages cache-related headers | Apiphany does not cache responses and manages cache-related headers</td>
</tr>
</tbody>
</table>

## Policy: Authenticate with Basic ##

**Description:**
Provide basic authentication credentials to the proxy for validation against the web service underlying this API.

**Policy Statement:**

	<authentication-basic username="username" password="password" />

**Example:**

> 		<inbound>
> 			<base />
> 			<authentication-basic username="administrator" password="admin123" />
 		</inbound>
 		<outbound>
 			<base />
 		</outbound>


**Where it can be applied:**
Use in the inbound section at API scope.

**When it should be applied:**
Enforcing access control to web resources

**Why it should be applied, why not:**
HTTP Basic Authentication (BA) implementation is the simplest technique for enforcing access controls to web resources because it doesn't require cookies, session identifier and log in pages. Rather, HTTP Basic authentication uses static, standard HTTP headers which means that no handshakes have to be done in anticipation. It is **recommended** that **SSL** be used for all communications so that no eavesdropper can discover the secret.

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>username="username"</td>
  <td>no restrictions, not case sensitive (ex: username="administrator")</td>
</tr>
<tr>
  <td>password="password"</td>
  <td>no restrictions on characters or symbol combinations, (ex: password="admin123")</td>
</tr>
</tbody>
</table>

## Policy: Set query string parameters ##

**Description:**
Query string parameter is a key/value pair of strings. A query string can also be additional information that a service may use to complete a request.

**Policy Statement:**

	<set-query-parameters>
 	<parameter name="param name" exists-action="override | skip | append | delete">
    </set-query-parameters>

**Example:**


> 		<policies>
  			<inbound>
>				<base />
    			<set-query-parameters>
      				<parameter name="api-key" exists-action="skip">
        			<value>4e2244001279ui7834a8cbe5cbdabba2a:16:67463271</value>
        			<!-- for multiple parameters with the same name add additional value elements -->
      				</parameter>
    			</set-query-parameters>
  			</inbound>
  		<outbound>
>				<base />
    	</outbound>
	</policies>


**Where it can be applied:**
Used anywhere in the inbound section at any scope.

**When it should be applied:**
Apply when one or more input parameters for a request are to be passed as part of the URL.

**Why it should be applied, why not:**
Use when passing input parameters of basic types (e.g. strings, integers) which are safe for transmission as plain text in the URL.
<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>parameter name="name"</td>
  <td>the key string to manage such as [param1]</td>
</tr>
<tr>
  <td>exists-action="override"</td>
  <td>takes the given entry [a=b&a=e] and passes an alternate entry [a=d]</td>
</tr>
><tr>
  <td>exists-action="skip"</td>
  <td>if entry [a=b]exist; everything remains unchanged</td>
</tr>
><tr>
  <td>exists-action="append"</td>
  <td>appends an additional entry to the end of the URL [a=b&a=e&c=d]</td>
</tr>
><tr>
  <td>exists-action="delete"</td>
  <td>removes entry[a=b] with this switch in affect you cannot declare a *value*</td>
</tr>
<tr>
  <td>value="value"</td>
  <td>value element [Parameter=value (Blue=3)]. For multiple parameters with the same name add additional value elements</td>
</tr>
</tbody>
</table>

## Policy: Convert JSON to XML ##

**Description:**
Converts a request or response body from JSON to XML.

**Policy Statement:**

	<json-to-xml apply="always | content-type-json" consider-accept-header="true | false"/>

**Example:**

	<policies>
       	<inbound>
            <base />
        </inbound>
    	<outbound>
			<base />
				<json-to-xml apply="always" consider-accept-header="false" />
    	</outbound>
	</policies>


**Where it can be applied:**
Use in the inbound or outbound sections at API or operation scopes.

**When it should be applied:**
When the content provider only sends JSON but the information is needed in XML format.

**Why it should be applied, why not:**
When a content provider drops support for XML and only supports JSON. If all of your apps were built on XML, your API will start to see HTTP error responses instead of XML responses, and so this policy can be used to convert responses to XML.

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>apply="always"</td>
  <td>Always converts Json to XML</td>
</tr>
><tr>
  <td>apply="content-type-json"</td>
  <td>If the content is Json then convert to XML</td>
</tr>
<tr>
  <td>consider-accept-header="true"</td>
  <td>will consider accept header</td>
</tr>
><tr>
  <td>consider-accept-header="false"</td>
  <td>will not consider accept header</td>
</tr>
</tbody>
</table>

## Policy: Convert XML to JSON ##

**Description:**
Converts request or response body from XML to either "JSON" or "XML faithful" form of JSON.

**Policy Statement:**

	<xml-to-json kind="javascript-friendly | direct" apply="always | content-type-xml" consider-accept-header="true | false"/>

**Example:**


> 	 	<policies>
       		<inbound>
            	<base />
        	</inbound>
    		<outbound>
				<base />
				<xml-to-json kind="direct" apply="always" consider-accept-header="false" />
    		</outbound>
		</policies>


**Where it can be applied:**
Use in the inbound or outbound sections at API or operation scopes

**When it should be applied:**
Provider drops XML

**Why it should be applied, why not:**
Your content provider sends JSON; your apps are built on XML to avoid getting HTTP error, API is converted to XML responses

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>kind="javascript-friendly | direct</td>
  <td>response body contains Javascript | response body contains XML</td>
</tr>
<tr>
  <td>apply= always | content-type-xml</td>
  <td>Always convert XML to JSON | If content is XML then convert to JSON</td>
</tr>
<tr>
  <td>consider-accept-header="true | false"</td>
  <td>will consider accept header | will not consider accept header</td>
</tr>
</tbody>
</table>

## Policy: Limit Call Rate ##

**Description:**
Prevents API usage spikes by limiting calls and/or the bandwidth consumption rate. 
**Response status:** *429 Too Many Requests*

**Policy Statement:**

	<rate-limit calls="number" renewal-period="seconds">
    <api name="name" calls="number" renewal-period="seconds">
    <operation name="name" calls="number" renewal-period="seconds" />
    </api>
    </rate-limit>

**Example:**

>		<policies>
    		<inbound>
                <base />
        		<rate-limit calls="20" renewal-period="90" />
    		</inbound>
    		<outbound>
           		<base />        
    		</outbound>
		</policies>


**Where it can be applied:**
Use in the inbound section at product scope.

**When it should be applied:**
When API calls have an impact on the set call limit (ex. 100 call limit)

**Why it should be applied, why not:**
In order to control the use of the API (the service which provides the 3rd party API applications) set a limit on how many times a call can be used in an hour. This limit applies to your API account rather than the applications which make the calls to the API i.e. you have 100 API calls per hour in total regardless of which applications is used - it is NOT 100 API calls per application. 

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>calls="number"</td>
  <td>the number of calls per API account</td>
</tr>
<tr>
  <td>renewal-period="seconds"</td>
  <td>number of seconds until the next renewable period, after exceeding the limit of API calls per account.</td>
</tr>
<tr>
  <td>api name="name"</td>
  <td>the name of the API that will be rate limited</td>
</tr>
<tr>
  <td>operation name="name"</td>
  <td>the name of the operation that will be rate limited</td>
</tr>
</tbody>
</table>

## Policy: Rewrite URI ##

**Description:**
Convert request URL from its public form to the form expected by the the web service.

**Policy Statement:**

	<rewrite-uri template="uri template" />

**Example:**


> 		<rewrite-uri template="getdata.xml" />


**Where it can be applied:**
Use anywhere in the inbound section at Operation scope only.

**When it should be applied:**

?

**Why it should be applied, why not:**

?

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>template="uri template"</td>
  <td>Content Cell</td>
</tr>

</tbody>
</table>

## Policy: Set usage quota ##

**Description:**
Enforce a renewable or lifetime call volume and / or bandwidth quota.

**Policy Statement:**

	<quota calls="number" bandwidth="kilobytes" renewal-period="seconds">
        <api name="name" calls="number" bandwidth="kilobytes" renewal-period="seconds">
        <operation name="name" calls="number" bandwidth="kilobytes" renewal-period="seconds" />
        </api>
        </quota>

**Example:**


> 		<quota calls="10000" bandwidth="40000" renewal-period="3600" />


**Where it can be applied:**
Use in the inbound section at Product scope.

**When it should be applied:**

?

**Why it should be applied, why not:**

?

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>calls="number"</td>
  <td>Content Cell</td>
</tr>
<tr>
  <td>bandwidth="kilobytes"</td>
  <td>Content Cell</td>
</tr>
<tr>
  <td>renewal-period="seconds"</td>
  <td>Content Cell</td>
</tr>
<tr>
  <td>api name="name"</td>
  <td>Content Cell</td>
</tr>
<tr>
  <td>operation name="name"</td>
  <td>Content Cell</td>
</tr>
</tbody>
</table>

## Policy: Set HTTP headers ##

**Description:**
Assign value to an existing or add a new response and / or request header.

**Policy Statement:**

	<set-headers>
        <header name="header name" exists-action="override | skip | append">
        <value>value</value> <!--for multiple headers with the same name add additional value elements-->
    	</header> 
		</set-headers>

**Example:**

> 		<set-headers>
        <header name="header name" exists-action="override | skip | append">
        <value>value</value> 
        </header>
        </set-headers>


**Where it can be applied:**
Use in the inbound and outbound sections at any scope, other than publisher.

**When it should be applied:**

?

**Why it should be applied, why not:**

?

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>header name="header name"</td>
  <td>Content Cell</td>
</tr>
<tr>
  <td>exists-action="override | skip | append"</td>
  <td>Content Cell</td>
</tr>
<tr>
  <td>value= "value"</td>
  <td>Content Cell</td>
</tr>
</tbody>
</table>

## Policy: Mask URLs in content ##

**Description:**
Use in the outbound section to re-write response body links and location header values making them point to the proxy.

**Policy Statement:**

	<redirect-body-urls />

**Example:**

> 		<redirect-body-urls />


**Where it can be applied:**
Apply at API or Operation scopes

**When it should be applied:**
For an opposite effect

**Why it should be applied, why not:**
Use in the outbound section for an opposite effect

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>redirect-body-urls</td>
  <td>Content Cell</td>
</tr>
</tbody>
</table>

## Policy: Restrict caller IPs ##

**Description:**
Allow calls only from specific IP addresses and / or address ranges. Forbid calls from specific IP address and / or address ranges.

**Policy Statement:**

	<ip-filter action="allow | forbid">
        <address>address</address>
        <address-range from="address" to="address" />
        </ip-filter>

**Example:**

> 		<ip-filter action="allow | forbid">
        <address>address</address>
        <address-range from="address" to="address" />
        </ip-filter>


**Where it can be applied:**
Use in the inbound section at any scope

**When it should be applied:**

?

**Why it should be applied, why not:**

?

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>ip-filter action="allow | forbid"</td>
  <td>Content Cell</td>
</tr>
<tr>
  <td>address="address"</td>
  <td>Content Cell</td>
</tr>
<tr>
  <td>address-range from="address" to="address"</td>
  <td>Content Cell</td>
</tr>
</tbody>
</table>

## Policy: Run custom script ##

**Description:**
Invoke specified JavaScript script to perform arbitrary request or response processing, e.g. content transformation, multiple API aggregation, API virtualization. 

**Policy Statement:**

	<run-script name="name" treat-as-response="true | false" timeout="seconds">
        <variables>
        <variable name="name">value</variable>
        </variables>
        </run-script>

**Example:**

> 		<run-script name="name" treat-as-response="true | false" timeout="seconds">
        <variables>
        <variable name="name">value</variable>
        </variables>
        </run-script>


**Where it can be applied:**
Use in the inbound and outbound sections at API or operation scopes.

**When it should be applied:**

?

**Why it should be applied, why not:**

?

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>name="name" treat-as-response="true | false" timeout="seconds"</td>
  <td>Content Cell</td>
</tr>
<tr>
  <td>variable name="name">value</td>
  <td>Content Cell</td>
</tr>
</tbody>
</table>


## Policy: Allow cross domain calls ##

**Description:**
Make the API accessible from Adobe Flash and Microsoft Silverlight browser-based clients.

**Policy Statement:**

	<cross-domain />

**Example:**

> 		<cross-domain />


**Where it can be applied:**
Use in the inbound section at publishers scope

**When it should be applied:**

? pending

**Why it should be applied, why not:**

? pending

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>cross-domain</td>
  <td>Content Cell</td>
</tr>
</tbody>
</table>

## Policy: Find and replace string in body ##

**Description:**
Find a request or response substring and replace it with a different substring.

**Policy Statement:**

	<find-and-replace from="what to replace" to="replacement" />

**Example:**

> 		<find-and-replace from="what to replace" to="replacement" />


**Where it can be applied:**
Use in the inbound and outbound sections at any scope.

**When it should be applied:**
At the Global level

**Why it should be applied, why not:**

?

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>from="what to replace"</td>
  <td>Content Cell</td>
</tr>
<tr>
  <td>to="replacement"</td>
  <td>Content Cell</td>
</tr>
</tbody>
</table>

## Policy: JSONP ##

**Description:**
Add support for JSONP to an operation or an API to allow cross-domain calls from JavaScript browser-based clients.

**Policy Statement:**

	 <jsonp callback-parameter-name="callback function name" />

**Example:**

> 	 	<jsonp callback-parameter-name="callback function name" />


**Where it can be applied:**
Use in the outbound section only.

**When it should be applied:**
To request data from a server in a different domain

**Why it should be applied, why not:**
JSONP or "JSON with padding" is a communication technique used in JavaScript programs which run in Web browsers. It provides a method to request data from a server in a different domain, something prohibited by typical web browsers because of the same origin policy. Under the same origin policy, a web page served from server1.example.com cannot normally connect to or communicate with a server other than server1.example.com.

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td>"callback function name"</td>
  <td>Content Cell</td>
</tr>
</tbody>
</table>

## Policy: CORS ##

**Description:**
CORS stands for cross-origin resource sharing. Add CORS support to an operation or an API to allow cross-domain calls from browser-based clients.

**Policy Statement:**

	 <cors>
        <allowed-origins>
        <origin>*</origin> <!-- allow any -->
        <!-- OR a list of one or more specific URIs (case-sensitive) -->
        <origin>URI</origin> <!-- URI must include scheme, host, and port. If port is omitted, 80 is assumed for http and 443 is assumed for https. -->
        </allowed-origins>
        </cors>

**Example:**

> 	 	<cors>
        <allowed-origins>
        <origin>*</origin> <!-- allow any -->
        <!-- OR a list of one or more specific URIs (case-sensitive) -->
        <origin>URI</origin> <!-- URI must include scheme, host, and port. If port is omitted, 80 is assumed for http and 443 is assumed for https. -->
        </allowed-origins>
        </cors>


**Where it can be applied:**
Use in the inbound section only.

**When it should be applied:**
To make XML http requests to another domain

**Why it should be applied, why not:**
CORS defines a way in which the browser and the server can interact to determine whether or not to allow the cross-origin request. It is more powerful than only allowing same-origin requests, but it is more secure than simply allowing all such cross-origin requests.

<table>
<thead>
<tr>
  <th>Statement</th>
  <th>Description</th>
</tr>
</thead>
<tbody>
<tr>
  <td><origin>*</origin></td>
  <td>allow any, OR a list of one or more specific URIs</td>
</tr>
><tr>
  <td><origin>URI</origin></td>
  <td>URI must include scheme, host, and port. If port is omitted, 80 is assumed for http and 443 is assumed for https.</td>
</tr>
</tbody>
</table>