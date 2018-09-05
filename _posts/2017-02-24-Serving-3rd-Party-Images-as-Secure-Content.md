---
layout: post
title: "Serving 3rd Party Images as Secure Content"
date: 2017-02-24 12:00:00
categories: [asp.net, how-to]
comments: false
---

I've built a site that's served over https, calling ASP.NET WebApi also over https, but contains images that are on plain old http. I trust these image locations, and just want to proxy them. This is kind of a wierd thing to want to do, but if this ever happens to you, here's how to do it:

**Step 1:** create a controller for the browser img tags to call

```csharp
[HttpGet]
[AllowAnonymous]
[CacheOutput(ServerTimeSpan = 3600, ClientTimeSpan = 3600)]
[Route(template: "BarSection/FooImage/{fooId}", Name = "FooImage")]
public async Task<HttpResponseMessage> FooImage(int fooId)
{
  var foo = await GetFooAsync(fooId);
  if (string.IsNullOrWhiteSpace(foo?.ImageUrl))
  {
    throw new HttpResponseException(HttpStatusCode.NotFound);
  }

  using (var client = new HttpClient())
  {
    var res = await client.GetAsync(paymentMethod.ImageUrl);
		
    var response = Request.CreateResponse(HttpStatusCode.OK);
		
	response.Content = new StreamContent(await res.Content.ReadAsStreamAsync());
	response.Content.Headers.ContentType = res.Content.Headers.ContentType;

	return response;
  }
}
```

**Step 2:** replace the urls in your models before providing them to the browser

```csharp
private string ProxyFooImageUrl(string externalUrl, int fooId)
{
  if (string.IsNullOrWhiteSpace(url))
  {
	return url;
  }

  var imageRoute = this.Url.Route("FooImage",
	new
	{
	  fooId = fooId,
	});

  // Force https, because even though the site is https the load balancer may be talking to this app via http
  var uri = new UriBuilder(new Uri(Request.RequestUri, imageRoute))
  {
  	Port = -1,
	Scheme = Uri.UriSchemeHttps
  }.Uri;

  return uri.AbsoluteUri;
}
```

**Step 3:** Profit


Note that it isn't a very good idea to serve arbitrary links from your server, so don't alter FooImage to directly accept links from the caller.

I happen to be using WebAPI, but I've added some MVC style caching using [Strathweb.CacheOutput](https://github.com/filipw/Strathweb.CacheOutput)