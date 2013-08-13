# Routing

- [Basic Routing](#basic-routing)
- [Route Parameters](#route-parameters)
- [Route Filters](#route-filters)
- [Named Routes](#named-routes)
- [Route Groups](#route-groups)
- [Sub-Domain Routing](#sub-domain-routing)
- [Route Prefixing](#route-prefixing)
- [Route Model Binding](#route-model-binding)
- [Throwing 404 Errors](#throwing-404-errors)
- [Routing To Controllers](#routing-to-controllers)

<a name="basic-routing"></a>
## Basic Routing 基本路由

Most of the routes for your application will be defined in the `app/routes.php` file. The simplest Laravel routes consist of a URI and a Closure callback.
你的应用程序的大部分路由都在app/routes.php文件中定义。最简单直接的Laravel路由，是由一个URI和一个封闭的回调函数组成，这种语法在Javascript中很常见。

以下是RESTful的路由定义实例：

**Basic GET Route** 基本的GET路由

	Route::get('/', function()
	{
		return 'Hello World';
	});

**Basic POST Route** 基本的POST路由

	Route::post('foo/bar', function()
	{
		return 'Hello World';
	});

**Registering A Route Responding To Any HTTP Verb** 注意一个可以响应所有http请求的路由

	Route::any('foo', function()
	{
		return 'Hello World';
	});

**Forcing A Route To Be Served Over HTTPS**  只有通过https才会响应的路由

	Route::get('foo', array('https', function()
	{
		return 'Must be over HTTPS';
	}));

Often, you will need to generate URLs to your routes, you may do so using the `URL::to` method:
由于有了上述的路由的定义，在程序的其他地方，可以使用`URL::to`方法产生url:

	$url = URL::to('foo');

<a name="route-parameters"></a>
## Route Parameters  路由的参数
	Route::get('user/{id}', function($id)
	{
		return 'User '.$id;
	});

**Optional Route Parameters**  可选的路由参数，这种方式的定义，可以允许提交的url中没有参数

	Route::get('user/{name?}', function($name = null)
	{
		return $name;
	});

**Optional Route Parameters With Defaults**   可选的路由参数，这种方式的定义，可以保证在没有提交参数name的情况下，程序有一个缺省的参数值

	Route::get('user/{name?}', function($name = 'John')
	{
		return $name;
	});

**Regular Expression Route Constraints**  通过正则表达式对提交的参数设定约束条件

	Route::get('user/{name}', function($name)
	{
		//
	})
	->where('name', '[A-Za-z]+');

	Route::get('user/{id}', function($id)
	{
		//
	})
	->where('id', '[0-9]+');

<a name="route-filters"></a>
## Route Filters  路由过滤器

Route filters provide a convenient way of limiting access to a given route, which is useful for creating areas of your site which require authentication. There are several filters included in the Laravel framework, including an `auth` filter, an `auth.basic` filter, a `guest` filter, and a `csrf`filter. These are located in the `app/filters.php` file.
过滤器提供了一个机制，可以方便的对一个给定的路由设定限制条件，这在开发人员需要为应用程序设定验证条件的时候非常有用。在Laravel框架中包含了一些预定义的过滤器，包括`auth`,`auth.basic`,`guest`和`csrf`4个过滤器，它们都可以在app/filters.php文件中看到。

**Defining A Route Filter**  定义一个路由过滤器

	Route::filter('old', function()
	{
		if (Input::get('age') < 200)
		{
			return Redirect::to('home');
		}
	});

If a response is returned from a filter, that response will be considered the response to the request and the route will not be executed, and any `after` filters on the route will also be cancelled.
如果一个响应是由过滤器返回的，那么包含这个过滤器的路由所指定的控制器或者回调函数就不会被继续执行，同时after过滤器程序也会被取消执行。

**Attaching A Filter To A Route**  给一个路由附加一个过滤器

	Route::get('user', array('before' => 'old', function()
	{
		return 'You are over 200 years old!';
	}));

**Attaching Multiple Filters To A Route**  给一个路由同时指定多个过滤器

	Route::get('user', array('before' => 'auth|old', function()
	{
		return 'You are authenticated and over 200 years old!';
	}));

**Specifying Filter Parameters**   给过滤器指定参数，可参考下面的实例代码

	Route::filter('age', function($route, $request, $value)
	{
		//
	});

	Route::get('user', array('before' => 'age:200', function()
	{
		return 'Hello World';
	}));

After filters receive a `$response` as the third argument passed to the filter:
过滤器可以接收一个response响应作为参数，那么这个$response会位于参数列表中的第3个位置

	Route::filter('log', function($route, $request, $response, $value)
	{
		//
	});

**Pattern Based Filters**   
路由的模式

You may also specify that a filter applies to an entire set of routes based on their URI.
也可以通过以下的方式为整个某个路由添加filter

	Route::filter('admin', function()
	{
		//
	});

	Route::when('admin/*', 'admin');

In the example above, the `admin` filter would be applied to all routes beginning with `admin/`. The asterisk is used as a wildcard, and will match any combination of characters.
在上面的例子中，admin过滤器会响应所有以admin/开始的路由。使用星号作为通配符，将匹配任何字符组合。

You may also constrain pattern filters by HTTP verbs:
您也可以通过HTTP的提交方式来作为过滤器的限制方式，比如下列post设定了过滤条件：

	Route::when('admin/*', 'admin', array('post'));

**Filter Classes**
通过面向对象的方式来设置过滤器

For advanced filtering, you may wish to use a class instead of a Closure. Since filter classes are resolved out of the application [IoC Container](/docs/ioc), you will be able to utilize dependency injection in these filters for greater testability.
为了实现更高级的功能，你可以通过定义一个自己的类取代回调函数方式的过滤器。这个类的对象要通过IoC容器注入到应用程序中，以减少程序之间的依赖关系。

**Defining A Filter Class**
定义一个过滤器类的的实例代码如下：
	class FooFilter {

		public function filter()
		{
			// Filter logic...
		}

	}

**Registering A Class Based Filter**
注册自定义的过滤器类

	Route::filter('foo', 'FooFilter');

<a name="named-routes"></a>
## Named Routes  给路由一个名字

Named routes make referring to routes when generating redirects or URLs more convenient. You may specify a name for a route like so:
命名一个路由，是为了在redirect和生成一个url的时候方便，如下面的实例

	Route::get('user/profile', array('as' => 'profile', function()
	{
		//
	}));

You may also specify route names for controller actions:
你也可以为自己的 控制器@方法 路由指定一个名字，如下列的例子：

	Route::get('user/profile', array('as' => 'profile', 'uses' => 'UserController@showProfile'));

Now, you may use the route's name when generating URLs or redirects:
现在，你就可以用下面的方式输出一个url或者重定向：

	$url = URL::route('profile');

	$redirect = Redirect::route('profile');

You may access the name of a route that is running via the `currentRouteName` method:
通过currentRouteName方法取得当前路由：

	$name = Route::currentRouteName();

<a name="route-groups"></a>
## Route Groups  路由组

Sometimes you may need to apply filters to a group of routes. Instead of specifying the filter on each route, you may use a route group:
定义一个路由组，可以方便的将该组路由分配一个过滤器，如下面的代码实例：

	Route::group(array('before' => 'auth'), function()
	{
		Route::get('/', function()
		{
			// Has Auth Filter
		});

		Route::get('user/profile', function()
		{
			// Has Auth Filter
		});
	});

<a name="sub-domain-routing"></a>
## Sub-Domain Routing  子域名的路由

Laravel routes are also able to handle wildcard sub-domains, and pass you wildcard parameters from the domain:
Laravel的路由器也可以通过通配符的实现子域名的适配：

**Registering Sub-Domain Routes**

	Route::group(array('domain' => '{account}.myapp.com'), function()
	{

		Route::get('user/{id}', function($account, $id)
		{
			//
		});

	});
<a name="route-prefixing"></a>
## Route Prefixing    路由前置符

A group of routes may be prefixed by using the `prefix` option in the attributes array of a group:
一个路由组，可以通过属性数组中设定prefix选项，来为prefix指定的名称设定路由组，如下面实例：

**Prefixing Grouped Routes**
路由组的前置选项
	Route::group(array('prefix' => 'admin'), function()
	{

		Route::get('user', function()
		{
			//
		});

	});

<a name="route-model-binding"></a>
## Route Model Binding
路由与Model对象的绑定

Model binding provides a convenient way to inject model instances into your routes. For example, instead of injecting a user's ID, you can inject the entire User model instance that matches the given ID. First, use the `Route::model` method to specify the model that should be used for a given parameter:
通过绑定，可以把整个model对象注入到路由的参数中。以实例说明，
 － 首先，绑定参数名到model类: Route::model('user', 'User');
 

**Binding A Parameter To A Model**

	Route::model('user', 'User');

Next, define a route that contains a `{user}` parameter:
－ 然后创建路由： Route::get( 'profile/{user}', function(User $user) {} );

	Route::get('profile/{user}', function(User $user)
	{
		//
	});

Since we have bound the `{user}` parameter to the `User` model, a `User` instance will be injected into the route. So, for example, a request to `profile/1` will inject the `User` instance which has an ID of 1.
－ 由于参数被绑定了，那么如果路径是 /profile/1， 那么laravel会把id为1的纪录提取出来并生成User对象注入到参数中

> **Note:** If a matching model instance is not found in the database, a 404 error will be thrown.
－ 如果没有找到制定id的纪录，那么laravel会抛出一个404错误。

If you wish to specify your own "not found" behavior, you may pass a Closure as the third argument to the `model` method:
－ 为了定制上述的缺省错误，给Route::model()方法第三个closure参数即可
	Route::model('user', 'User', function()
	{
		throw new NotFoundException;
	});

Sometimes you may wish to use your own resolver for route parameters. Simply use the `Route::bind` method:
最后是可以自己完全的定制绑定的参数的类型，采用Route::bind() 方法

	Route::bind('user', function($value, $route)
	{
		return User::where('name', $value)->first();
	});

<a name="throwing-404-errors"></a>
## Throwing 404 Errors  抛出404错误

There are two ways to manually trigger a 404 error from a route. First, you may use the `App::abort` method:
有2种手动的方式从路由种触发404错误，首先，你可以使用App::abort
	App::abort(404);

Second, you may throw an instance of `Symfony\Component\HttpKernel\Exception\NotFoundHttpException`.
第二个方式，你可以抛出一个Symfony\Component\HttpKernel\Exception\NotFoundHttpException对象的实例。


More information on handling 404 exceptions and using custom responses for these errors may be found in the [errors](/docs/errors#handling-404-errors) section of the documentation.
更多关于如何处理404异常，和使用自定义的响应处理程序的错误，可以到/docs/errors#handling-404-errors文件中查看更多细节。

<a name="routing-to-controllers"></a>
## Routing To Controllers  路由到控制器

Laravel allows you to not only route to Closures, but also to controller classes, and even allows the creation of [resource controllers](/docs/controllers#resource-controllers).
Laravel允许你不光使用回调函数来处理路由，而且也可以路由到你的控制器类，甚至允许到resource控制。请参考控制器的有关章节。

See the documentation on [Controllers](/docs/controllers) for more details.
