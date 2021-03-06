# Laravel Inspector
Laravel 5.x package to inspect, debug and profile your server side code using your browser's console.

* [At a Glance](#glance)
* [Installation](#installation)
* [Configuration](#configuration)
* [Usage](#usage)
* [Messages](#messages)
* [Timers](#timers)
* [Conditionals](#conditionals)
* [Redirects](#redirects)
* [Dump and die](#dump)
* [Exceptions](#exceptions)
* [VIEW/AJAX/API requests, how it works](#requests)
* [License](#license)


## <a name="glance"></a>At a Glance


|[![](https://s31.postimg.org/d5y10qy57/image.png)](https://s31.postimg.org/xdbgt1vmh/image.png)|[![](https://s31.postimg.org/vo2dkzfx7/image.png)](https://s31.postimg.org/hhmmpr521/image.png)|
|:-:|:-:|
|Messages|Exceptions|

|[![](https://s31.postimg.org/kw07s5b2z/image.png)](https://s31.postimg.org/hcea2c8d5/image.png)|[![](https://s31.postimg.org/o49aixmqj/image.png)](https://s31.postimg.org/8vjd55t21/image.png)|
|:-:|:-:|
|idd() - Dump and die on steroids|idd() SQLs page|

|[![](https://s31.postimg.org/g1b47m257/image.png)](https://s31.postimg.org/7vt29gdw9/image.png)|[![](https://s31.postimg.org/ps8fxl0m3/image.png)](https://s31.postimg.org/4vc7sx2l5/image.png)|
|:-:|:-:|
|Laravel Inspector as the default Exception renderer|Timers and Timeline|


|[![](https://s31.postimg.org/uaaqpognv/image.png)](https://s31.postimg.org/3p87u4eah/image.png)|[![](https://s31.postimg.org/vduv1n1az/image.png)](https://s31.postimg.org/ke9nq1avt/image.png)|
|:-:|:-:|
|Redirection|API/Ajax calls|

|[![](https://s31.postimg.org/7be16rknv/image.png)](https://s31.postimg.org/inqmojtcp/image.png)|[![](https://s31.postimg.org/nqy57iynf/image.png)](https://s31.postimg.org/gayvlqay1/image.png)|
|:-:|:-:|
|Using Postman REST client app|<code>laravel_ispector=dump</code> parameter|


|Available Collectors|Information about|
|---|---|
|MessageCollector|User's messages and dumps|
|ExceptionCollector|Exceptions|
|DBCollector|Queries, including execution time and parameters binding|
|TimersCollector|Timers and time stamps|
|RoutesCollector|Application routes|
|RequestCollector|Current Request|
|ResponseCollector|Current Response|
|SessionCollector|Session variables|
|ServerCollector|$_SERVER dump|
|More to come...| |


[comment]:$  
## <a name="installation"></a>Installation

**The package was developed using PHP 7 so there may be some -minor- incompatibilities with PHP 5.4** 

Installing the package via composer:

	composer require lsrur/inspector


Next, add InspectorServiceProvider to the providers array in <code>config/app.php</code>:

	Lsrur\Inspector\InspectorServiceProvider::class,

  
And this Facade in the same configuration file:

	'Inspector' => Lsrur\Inspector\Facade\Inspector::class,
	


## <a name="configuration"></a>Configuration

In order to use Inspector as the default exceptions renderer, add the following line in the file <code>app/Exceptions/Handler.php</code> of your Laravel project:

```php    
    public function render($request, Exception $e)
    {
        \Inspector::renderException($e); 	// <= THIS LINE
        return parent::render($request, $e);
    }  
```      

  
## <a name="usage"></a>Usage
Laravel inspector can be invoked using the Facade, the provided helper functions and a Blade directive:

```php	
	//Using Facade
	\Inspector::log(...);
	\Inspector::info(...);
		
	//Using the "inspector" helper function
	inspector()->info(...);
	inspector()->warning($myVar);
	
	// "li" is an alias of "inspector"
	li()->error(...);
	li()->group('myGroup');
	
	// "inspect" function makes an "Inspector::log($v)" for each passed argument
	inspect($var1, $var2, $var3, ...);
	
	// Dump and die using Laravel Inspector magic
	idd();
	idd($var1, $var2);
	
	// Inside Blade
	@li(cmd,param1,param2,...)
	
	// samples
	@li('log', 'My comment', $myVar)
	@li(log, 'My comment', $myVar) //also works with no command quotes
	@li(group,"myGroup")
	@li(endGroup)
	
```	

**Laravel inspector will only be active if the config variable <code>app.debug</code> is true.**  
Anyway, you can temporarily turn Inspector off (just for the current request) with:

	li()->turnOff();

## <a name="messages"></a>Messages
You can inspect objects and variables with the following methods, each of which has its own output format:

|Method|Description |
|----|-----|
|<code>log([string  $description,] mixed $data)</code>|Outputs data with "log" format |
|<code>info([string $description,] mixed $data)</code>|Outputs data with "info" format |
|<code>error([string $description,] mixed $data)</code>|Outputs data with "error" format |
|<code>warning([string $description,] mixed $data)</code>|Outputs data with "warning" format |
|<code>success([string $description,] mixed $data)</code>|Outputs data with "success" format |
|<code>table([string $description,] mixed $data)</code>|Outputs data inside a table |

[comment]: $

Examples:

```php	
	li()->log("MyData", $myData);
	li()->info($myData);
	li()->table("clients", $myClientsCollection); 
```
Additionally, you can use the "inspect" helper function to quickly inspect objects and variables.

```php	
	inspect($var1, $var2, $var3,...);

```

#### Grouping Messages
Laravel Inspector allows you to group messages into nodes and subnodes:

```php	
	li()->group('MyGroup');
		li()->info($data);
		li()->group('MySubGroup');
			li()->error('oops', $errorCode);
		li()->groupEnd();
		li()->success('perfect!');
	li()->groupEnd();
```		
In addition to the ability to group information, each group and subgroup excecution time will be measured and shown. If you forget to close a group, Laravel Inspector will automatically do it at the end of the script, but the excecution time for that group can not be taken.

## <a name="timers"></a>Timers

|Method|Description |
|----|-----|
|time(string $timerName)|Starts a timer |
|timeEnd(string $timerName)|Ends a timer |
|timeStamp(string $name)|Adds a single marker to the timeline |

[comment]: $

Examples:

```php	
	li()->timer("MyTimer");
	// ...
	li()->timeEnd("MyTimer");
	
	li()->timeStamp('Elapsed time from LARAVEL_START here'); 
```

## <a name="conditionals"></a>Conditionals
Laravel Inspector includes a handy method <code>if()</code>to operate conditionally:

```php	
	li()->if($count > 100)->warning('too much?', $count);
	
	li()->if($itsHappeningAgain)->time('myTimer');
	// ...
	li()->if($itsHappeningAgain)->timeEnd('myTimer');
	// OR
	li()->timeEnd('myTimer'); // do not worry if it has not been initialized
	
	// Dump and die
	li()->if(!$whatIExpect)->dd($thatThing);
	
```
The <code>if()</code> function is not avalilable under the blade directive <code>@li</code>.

## <a name="redirects"></a>Redirects
Laravel Inspector handles redirects smoothly; showing the collectors bag for both, the original and the target views.

## <a name="dump"></a>Dump and die

The <code>dd()</code> method (or <code>idd()</code> helper) will dump the entire collectors bag and terminates the script: 

```php	
	\Inspector::dd();
	li()->dd();
	
	// or simply
	idd();
	
	// adding last minute data
	idd($var1, $var2,...)

	
```
As the rest of the package, this feature intelligently determines how will be the format of the output, even if the call was performed from CLI. 

Another way to make an inspection, but without interrupting the flow of the request/response, is by adding the parameter <code>laravel_inspector=dump</code> to the URL:
  
<code>http://myapp.dev/contacts?id=1&laravel_inspector=dump</code>
 
Thus, Laravel Inspector wont be activated until the a terminable middleware is reached.

## <a name="exceptions"></a>Exceptions

The function <code>addException()</code> will inspect our caught exceptions:  

```php	
	try {
		...
	} catch (Exception $e) {
		li()->addException($e);
	}

```

Optionally, you can setup LI as the default exception renderer during development time (app.debug=true). Refer to the [configuration options](#config) to do so. 

## <a name="requests"></a>VIEW/AJAX/API requests, how it works 
 
Laravel Inspector (LI) automatically detects the type of the request/response pair and determines the output format. If a View response is detected, the code needed to elegantly show the collected information in the browser console will be injected as a javascript into that view. Along with this, LI will also add a small piece of pure javascript code that serves as a generic http interceptor, which will examine subsequent AJAX calls looking for information injected by LI
(this interceptor was tested under pure javascript, Angular 1.x ($http) and jQuery ($.ajax) and should work with any js framework). The interceptor also adds a header in each client AJAX call to let LI  know that the interceptor is present. Then, from Laravel side, during an AJAX request or a JSON response, LI will send a script to be interpreted (and properly rendered in the browsers console) by the interceptor, OR a pure Json if that header is not present and then assuming that the request was sent from cURL, a REST client app or something else.

If you are developing, for example, an SPA and using Laravel only for the API but not to serve the web page/s, you can include the following code in your client app to take full advantage of all formatting features of Laravel Inspector.

```javascript
(function(XHR) {
	"use strict";

	var send = XHR.prototype.send;

	XHR.prototype.send = function(data) {
		var self = this;
		var oldOnReadyStateChange;
		var url = this._url;
		this.setRequestHeader('Laravel-Inspector', 'interceptor-present');
		function onReadyStateChange() {
			if(self.readyState == 4 /* complete */) {
				var response = JSON.parse(this.response);
				if (typeof response.LARAVEL_INSPECTOR !== 'undefined') {
					if(typeof response.LARAVEL_INSPECTOR === 'string')
					{
						eval(response.LARAVEL_INSPECTOR);
					} else {
						console.log('LARAVEL INSPECTOR ', response);
					 }
				 }   
			}
			if(oldOnReadyStateChange) {
				oldOnReadyStateChange();
			}
		}
		if(!this.noIntercept) {            
			if(this.addEventListener) {
				this.addEventListener("readystatechange", onReadyStateChange, false);
			} else {
				oldOnReadyStateChange = this.onreadystatechange; 
				this.onreadystatechange = onReadyStateChange;
			}
		}
		send.call(this, data);
	}
})(XMLHttpRequest);
```
 
[comment]: $
    
## <a name="license"></a>License
Laravel Inspector is licensed under the [MIT License](https://opensource.org/licenses/MIT).