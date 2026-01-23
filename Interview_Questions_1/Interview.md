**.net core vs frame work**

1> cross platform

2> Open source

3> Integration of UI

4> Hosting -- Kestrel , IIS , nginx.

5> Dependency Injection

6> Supports Multiple IDE


**.Net Request LifeCycle:**


call the request from the client

host server creation

server fetches the request and pass it to middleware pipeline

then routing will happen

model binding

Action Execution

result execution

response middleware throw the result to client.



**Kestrel Working**



Sockets Listening : Kestrel opens TCP sockets on configured ports (e.g., 5000, 443).

Connections Management : Each Connection is established and handled independently

Request Parsing : Raw Bytes which are request header , request line and body are parsed.

httpContext Creation : http response and http request Object creation takes place.

middleware execution

controller end point execution

Connection Reuse



**HostBuilder vs WebApplicationBuilder?**



Host Builder is Generic Hosting Model used for Different Type of Applications (Console, Web apps , Worker and background services) introduced in .NET core 2.1

WEbApplicationBuilder is minimal hosting model for web applications (web api , mvc , minimal apis and razor pages) introduced in .NET 6



**What are SDK-style projects?**



new project file structure introduced in .NET core. majority of SDK defaults were already defined which are not included in project file.



**JIT vs AOT** :: compile at runtime, compile at Build time.



**How does Side-by-Side versioning work in .NET Core?**



each app will run individually with different runtimes as mentioned in app.runtimeconfig.json

app runs on its own process.



**What is self-contained vs framework-dependent deployment?**



self-contained : app will be deployed with its runtime. unlike FDD



**Explain Dependency Injection in ASP.NET Core.**



Dependency Injection (DI) is a design pattern used to provide required objects (dependencies) to a class instead of creating them inside the class.

public IActionResult Get(\[FromServices] IMyService service) --> Method Injection.



Note::

DbContext must always be Scoped, never Singleton → prevents thread conflicts.

Transient can be injected into Singleton → creates one instance per injection.

Singleton objects live for app lifetime, so avoid storing request-specific data inside them



**What happens if a Scoped service is injected into Singleton?**


Will give runtime error -- If the Singleton held a reference to the Scoped service:

The Scoped service might point to a disposed object after the request ends

Leads to bugs, memory leaks, or unexpected behavior.

Instead we can create scoped service on demand using(var scope = provider.CreateScope()){}


**Difference between Use, Run, and Map?**

Use ==> Adds middleware to the pipelines, pass the control forward.

Run ==> will add terminal middleware to pipeline, generate response.

Map ==> will create a branch which satisfy the specified Path.


**How do you write custom middleware?**

Inherit customMiddleware

Inject requestDelegate 

create invokeAsync(HttpContext context) method



app.UseMiddleware<MyCustomMiddleware>();

What is IHostedService?

Its allow us to run the background code as part of applications Lifetime.

public interface IHostedService
{
    Task StartAsync(CancellationToken cancellationToken);
    Task StopAsync(CancellationToken cancellationToken);
}

IHostedService vs BackgroundService

IHostedService → low-level, you manage everything

BackgroundService → built-in base class that already implements IHostedService (recommended for long-running tasks)


Difference between Filters vs Middleware.

They work in similar way but filters execute when the control reaches to controller,only applies to requests of controller.while middleware is part of request pipeline applies to every request. 

public class LogActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context) { }
    public void OnActionExecuted(ActionExecutedContext context) { }
}

[ServiceFilter(typeof(LogActionFilter))]
public IActionResult Index() => Ok();


Difference between MVC and Minimal APIs.

app.MapGet("/products/{id}", (int id) =>
{
    return Results.Ok(new { Id = id });
});

MVC --More Boiler Plate , More time.
Minimal API -- Light Weight, used in Building Microservices.

How does Model Binding work internally?

Http Requests --> Value Providers --> Model Binder Providers --> Model Binders --> Actions Parameters.

Value Providers :: extract data from specific part of http request.
Model Binder Providers :: checks the type of the Model and decides whether it can handle request r not. 

SimpleTypeModelBinder → int, string
ComplexTypeModelBinder → objects
CollectionModelBinder → lists
BodyModelBinder → request body (JSON)

Model Binders :: 
It requests values from value providers
Build the Object graph
Handles nested properties
Track Binding errors.

Validation :: 
Validation attributes were evaluated (modelState.isValid);

public class CustomBinder : IModelBinder
{
    public Task BindModelAsync(ModelBindingContext context)
    {
        var value = context.ValueProvider.GetValue("custom").FirstValue;
        context.Result = ModelBindingResult.Success(value);
        return Task.CompletedTask;
    }
}


What is Model Validation?
is a process of Validating the incoming request.Before action logic runs.
1> Attribute Validation(Data Annotation) ([Required],[RegularExpression])
2> Fluent Validation (RuleFor(x => x.Email).NoEmpty()) ==> External Library
3> Custom Validation 

public class OrderDto : IValidatableObject
{
    public IEnumerable<ValidationResult> Validate(ValidationContext context)
    {
        if (Total <= 0)
            yield return new ValidationResult("Total must be positive");
    }
}

How do you handle global exception handling?

Its a way of handling unknown errors arises from api, helps in to avoid leakage, and not exposing data to the user.

app.UseExceptionHandler("/error");

app.Map("/error", (HttpContext context) =>
{
    var exception = context.Features.Get<IExceptionHandlerFeature>()?.Error;

    return Results.Problem(
        title: "An error occurred",
        statusCode: StatusCodes.Status500InternalServerError
    );
});

What are Action Filters, Result Filters, Exception Filters?(Filters are only in MVC)

Action Filters :: Execute when before and after the action method calls.
Result Filters :: execute after and before the action result is being called.
Exception Filter :: execute when action and result execution fails.


What is Attribute Routing vs Conventional Routing?

AttributeRouting :: Use attributes at controller and actionMethod Level ([Route("GetData")])
Conventional Routing :: Declare the routes using patterns in program.cs 
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

Note:: We can use both at same time.

Route precedence :: 

1> attribute Routes take precedence Over conventional Routes.
2> More specific routes have higher precendence 
[HttpGet("products/new")]       // literal → higher precedence
[HttpGet("products/{id}")]      // parameter → lower precedence
3> Order Property set the precedence
[Route("api/products"), Order = 1]  // Higher priority
[Route("api/{controller}"), Order = 2]
4> constraints also affect precedence

advanced constraints ::
advanced constraints allows us to restrict the parameters used in routing 
{date:datetime}
{name:max(10)}
{slug:regex(^[a-z0-9-]+$)}
{id:required}

How does API Versioning work?

It will be helpfulwhen user wants to use old version and some breaking changes going to be deployed.
Breaking changes ::: Rename Fields, change response structure, change Business Logic.
Non Braking changes ::: add new API , add optional parameters , 

1> URL Path Versions 
2> Query Parameter Versioning
3> Header based Versioning 
4> Media Type Versioning

Install Dependency asp.versioning.mvc.apiexplorer
[Route("api/v{version:api-version}/{controller}"]
[apiVersion("1.0")]

Builder.Service.AddApiVersioning(
options.ApiVersionReder = new QueryStrinApiVersionReader('api-version');
or
options.ApiVersionReder = new HeaderApiVersionReader('x-api-version');
or
options.ApiVersionReder = new MediaTypeApiVersionreader('api-version');
).addmvc().addapiExplorer(
opt => opt.GroupNameFormat = "'v'V'");

How do you implement Swagger customization?

Install package swashbuckle.aspnetcore
need to enable xmldocumentation and xmlcomments ==> properties --> Build --> Output --> check Documentation File
or 
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>  ==> oin csproj file
  <NoWarn>$(NoWarn);1591</NoWarn>
</PropertyGroup>

services.AddSwaggerGen( opt => {
opt.IncludeXmlComments(Path.Combine(AppCotext.BaseDirectory, $"{Assembly.GetExecutingAssemby().Getname().Name}.xml"))  // to include the comments in swagger.
});



c.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme
    {
        Name = "Authorization",
        Type = SecuritySchemeType.Http,
        Scheme = "Bearer",
        BearerFormat = "JWT",
        In = ParameterLocation.Header,
        Description = "Enter 'Bearer' [space] and then your token.\n\nExample: Bearer eyJhbGciOiJIUzI1NiIs..."   // to setup JWT token
    });

    c.AddSecurityRequirement(new OpenApiSecurityRequirement
    {
        {
            new OpenApiSecurityScheme
            {
                Reference = new OpenApiReference
                {
                    Type = ReferenceType.SecurityScheme,
                    Id = "Bearer"
                }
            },
            Array.Empty<string>()
        }
    });


How do you improve ASP.NET Core performance?

1> Hosting Services --> Prefer Kestrel + Reverse Proxy
2> Enable Http/2, Http/3
Builder.WebHost.ConfigureKestrel(Options => {
options.ListenAnyIp(5001, opt => {
opt.useHttps();
opt.protocols = HttpProtocols.Http2;
});
});

3>Use Asynchronous Programmming correctly.
var data = httpClient.GetStringAsync(url).Results // Bad

4> Reduce MiddleWare & Pipeline Cost  // Remove unused Middle wares

5> Use caching Aggressively
Builder.Services.AddMemoryCache();
var results = await _cache.GetOrCreateAsync("key", entry=>{
entry.AbsoluteExpirationRelativetoNow = TimeSpan.FromMinutes(5);
return GetDateFromDbAsync();
});

6> Select Only needed columns  // Select(u => new {u.Id, u.Name})

7> Use Compiled queries for hot paths. // getUser = EF.CompileAsyncQuery(appdbcontext ctx,int id) => ctx.Users.FirstOrDefault(u => u.Id == id));

8> Response Compression // builder.Services.AddresponseCompression();  app.UseResponseCompression();

9> Reuse HttpClient // builder.Services.AddHttpClient();
since each client creates its own socket and DNS changes were happens.

.AddTransientHttpErrorPolicy(p =>
        p.WaitAndRetryAsync(3, _ => TimeSpan.FromSeconds(2))); // for adding retry policy in case of errors

10> Minimise Json Serialisation Cost.

Use System.Text.Json  // since it is built in
Use Source Generators (High performance)  //[JsonSerializable(typeof(MyDto))]
public partial class AppJsonContext : JsonSerializerContext { }

11> Enable Output Caching 

builder.Services.AddOutputCache();
app.UseOutputCache();

[OutputCache(Duration = 60)]
public IActionResult Get() 
{ 
    return Ok(data);
}

 // mechanism that let us store the response and serves it for subsequent requests
note ==> OutputCache doesn't executes endpoints unlike responseCache.

12> Use BackGround Services for Heavy Work 

13> Enable Health Checks.

What is response caching?

[ResponseCache(Duration = 60)]














Angular ::::

Types of templates in angular

Inline Templates :: HTML directly written in the component (Template : '<b>jbkdbm</b>',)
External Template Files : link the html file in the component (templateUrl : './example.html',)




