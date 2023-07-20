# middleware

Code Step by Step -> Anil Sidhu

Laravel includes a middleware that verifies the user of your application is authenticated. 
If the user is not authenticated, the middleware will redirect the user to your application's login screen. 
However, if the user is authenticated, the middleware will allow the request to proceed further into 
the application that is  it redirects to the home page.

Middleware acts as a bridge between a request and a response. It is a type of filtering mechanism.

location: app/Http/Middleware

Registering Middleware
1. Global Middleware
2. Grouped Middleware
3. Routed Middleware

**** Global Middleware

1. laravel new middleware_practice
2. php artisan serve
3. php artisan make:middleware checkAge
4. users.blade.php 
	<h1>Welcome to the website</h1>
	
5. noaccess.blade.php
	<h1>You Can't Access our website</h1>
	
6. /Middleware/checkAge.php
	public function handle(Request $request, Closure $next)
    {
        echo "Global Message";
        if($request->age && $request->age<18){
            return redirect('noaccess');
        }
        return $next($request);
    }
	
7. Kernel.php
	 protected $middleware = [
        \App\Http\Middleware\checkAge::class,
        \App\Http\Middleware\TrustProxies::class,
        \Illuminate\Http\Middleware\HandleCors::class,
    ];
	
8. web.php
	Route::view("users","users");
	Route::view("noaccess","noaccess");

9. http://127.0.0.1:8000/users?age=20 - Global Message -----> Welcome to the website
	http://127.0.0.1:8000/users?age=15 --> http://127.0.0.1:8000/noaccess ---> Global Message -----> You Can't Access our website
	
**** Group Middleware

1. php artisan make:middleware groupcheckAge
2. home.blade.php
	<h1>Home Page</h1>
3. Kernel.php
	protected $middlewareGroups = [
		'api' => [
            // \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
            'throttle:api',
            \Illuminate\Routing\Middleware\SubstituteBindings::class,
        ],
        'ageChecker'=>[
            \App\Http\Middleware\groupcheckAge::class,
        ],
    ];
4. groupcheckAge.php
	public function handle(Request $request, Closure $next)
    {
        if($request->age && $request->age<18){
            return redirect('noaccess');
        }
        return $next($request);
    }

5. web.php
	Route::view("home","home");

	Route::view("noaccess","noaccess");

	Route::group(['middleware'=>['ageChecker']], function(){
		Route::view("users","users");
		Route::get('/', function () {
        return view('welcome');
    });
	});
	
6. http://127.0.0.1:8000/home?age=2
   http://127.0.0.1:8000/home?age=23
   
   http://127.0.0.1:8000/users?age=2 --> http://127.0.0.1:8000/noaccess
   http://127.0.0.1:8000/users?age=23 
   
   http://127.0.0.1:8000/?age=2  --> http://127.0.0.1:8000/noaccess
   http://127.0.0.1:8000/users?age=23 
   

**** Route Middleware
1. php artisan make:middleware routecheckAge

2. Kernel.php
	
   protected $routeMiddleware = [
        'routecheckAge' => \App\Http\Middleware\routecheckAge::class,
        'auth' => \App\Http\Middleware\Authenticate::class,
    ];

3. routecheckAge.php
	public function handle(Request $request, Closure $next)
    {
        if($request->age && $request->age<18){
            return redirect('noaccess');
        }
        return $next($request);
    }

4. Route::view("users","users")->middleware('routecheckAge');

5. http://127.0.0.1:8000/?age=2
	http://127.0.0.1:8000/users?age=23
	http://127.0.0.1:8000/users?age=2
