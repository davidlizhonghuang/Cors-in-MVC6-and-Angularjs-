# Cors in MVC6 and Angularjs
<pre>
Introduction

we know OWIN asp.net Web API 2 has Cors support that can fix developer's headache Cors problems in MVC when 
developers develop clients to access the web service in another domain. The real world contains a lot of MVC
API controller based resources that need to be accessed by different clients. Therefore, we need to know how 
to configure Cross Domain Allow Origin : * in MVC 6 and Angular JS http post actions.  

Get Started

1, MVC 6 API (cross domain)

VS 2015 can create a brand new MVC 6 template. We can easily get API controller worked here. In order to 
enable the "cross domain allow origin":*, we can simply update the startup.cs file as below 

1, add Microsoft.AspNet.Cors to project.json  

2, add the following code in ConfigureServices method
     
 public void ConfigureServices(IServiceCollection services)
 {
            var policy = new Microsoft.AspNet.Cors.Infrastructure.CorsPolicy();

            policy.Headers.Add("*");
            
            policy.Methods.Add("*");
            
            policy.Origins.Add("*");
            
            policy.SupportsCredentials = true;

            services.AddCors(xx => xx.AddPolicy("mypolicy", policy));
            //services.AddCors();
	.......
}

3,  add the following code in configure method

 public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
 
  app.UseCors("mypolicy");
...
}

That is it. this resource is ready to be accessed by cross domain clients. However, Clients needs to have
a correect header before they can be allowed to access this resource. We use Angular JS client http service 
to call this resource,we need to add some headers in angular http service.  

Cors in Angular JS (origin)

It is simple to configure the $http service in angular js as Cors enabled as below

 cors.config(function ($httpProvider) {   //for POST

           $httpProvider.defaults.headers.common = {};
           $httpProvider.defaults.headers.post = {};
           $httpProvider.defaults.headers.put = {};
           $httpProvider.defaults.headers.patch = {};
         
           $httpProvider.defaults.headers.post['Content-Type'] = 'application/x-www-form-urlencoded';
})

when we do http.post call, server side will do the preflight check, we can reset headers to ignore this check 
and then give the correct content type to header, post normally means we need to submit the data in html form 
to server, the content typevshould be the 'application/x-www-form-urlencoded', it can not be the 'applcaiton/json'
or it will be fail to do cross domain call.

After we get the correct content type for header,  we use example method below in angular js to call this resource

 		$http({
                   method: 'POST',
                   url: "http://mvc6/webapicontroller/action",
                   data:  JSON.stringify($scope.formdata) 
                }).success(function (response) {
                      $scope.message = response;
                   })
                   ;

we pass $scope.formdata form data as json string to api controller, api controller in another web site will pick up 
this form data as code below

[httppost]
public IActionresult Post(){ 

 	   var ss = Request.Form.Keys.ToList(); //server can see the request data ,

            Author m = JsonConvert.DeserializeObject<Author>(ss[0]);

            var uthor = new Author();

            uthor.AuthorID = 0;
            uthor.FirstMidName = m.FirstMidName;
            uthor.LastName = m.LastName;

            if (!ModelState.IsValid)
            {
                return HttpBadRequest(ModelState);
            }

            _context.Authors.Add(uthor);

            try
            {
                await _context.SaveChangesAsync();
            }
            catch (DbUpdateException ex)
            {
                if (AuthorExists(uthor.AuthorID))
                {
                    return new HttpStatusCodeResult(StatusCodes.Status409Conflict);
                }
                else
                {
                    var err= ex.Message;
                    throw;
                }
            }

            return CreatedAtRoute("GetAuthor", new { id = 1  }, author);

} 

API controller POST method can see the request info from remote client, this is important , we can simply extract the 
submitted data from this request object for data insert.

Summary

Besides OWIN Asp.Net Webapi 2 that is Cors suppot enabled, we can also configue the Cors in MVC 6 and angular js front 
end to make mutual communications possible. 

</pre>
 
