# TinyHttp
TinyHttp make easier the dialog between your API and your application.
It hide all the complexity of communication, deserialisation ...

[![Build status](https://ci.appveyor.com/api/projects/status/08prv6a3pon8vx86?svg=true)](https://ci.appveyor.com/project/jgiacomini/tinyhttp)

## Nuget
* Available on NuGet: [Tiny.Http](http://www.nuget.org/packages/Tiny.Http) [![NuGet](https://img.shields.io/nuget/v/Tiny.Http.svg?label=NuGet)](https://www.nuget.org/packages/Tiny.Http/)

## Platform Support
|Platform|Supported|Version|
| ------------------- | :-----------: | :------------------: |
|.Net Standard|Yes|2.0|


## Features
* Modern async http client for REST API.
* Support of verbs : GET, POST , PUT, DELETE, PATCH, HEAD
* Support of custom http verbs
* Support of cancellation token on each requests
* Automatic XML and JSON serialization / deserialization
* Support of custom serialisation / deserialisation
* Support of multi-part form data
* Download file
* Upload file
* Optimized http calls
* Typed exceptions which are easier to interpret
* Provide an easy way to log : all sending of request, failed to get response,  and the time get response.

## Basic usage

### Create the client
```cs
using Tiny.Http;

var client = new TinyHttpClient("http://MyAPI.com/api", new HttpClient());
```

### Headers

#### Default header for all requests

```cs
// Add default header for each calls
client.DefaultHeaders.Add("Token", "MYTOKEN");
```
#### Add header for current request


```cs
// Add header for each calls
client.GetRequest("City/All").
      AddHeader("Token", "MYTOKEN").
      ExecuteAsync();
```

#### Read headers of response

```cs
Headers headersOfResponse = null;
await client.GetRequest("GetTest/HeadersOfResponse").
             FillResponseHeaders(out headersOfResponse).
             ExecuteAsync();
foreach(var header in headersOfResponse)
{
    Debug.WriteLine($"{current.Key}");
    foreach (var item in current.Value)
    {
        Debug.WriteLine(item);
    }
}
```

### Basic GET http requests

```cs
var cities = client.GetRequest("City/All").ExecuteAsync<List<City>>();
// GET http://MyAPI.com/api/City/All an deserialize automaticaly the content

// Add a query parameter
var cities = client.
    GetRequest("City").
    AddQueryParameter("id", 2).
    AddQueryParameter("country", "France").
    ExecuteAsync<City>> ();

// GET http://MyAPI.com/api/City?id=2&country=France an deserialize automaticaly the content

```

### Basic POST http requests
```cs
// POST
 var city = new City() { Name = "Paris" , Country = "France"};

// With content
var response = await client.PostRequest("City", city).
                ExecuteAsync<bool>();
// POST http://MyAPI.com/api/City with city as content

// With form url encoded data
var response = await client.
                PostRequest("City/Add").
                AddFormParameter("country", "France").
                AddFormParameter("name", "Paris").
                ExecuteAsync<Response>();
// POST http://MyAPI.com/api/City/Add with from url encoded content


var fileInfo = new FileInfo("myTextFile.txt");
var response = await client.
                PostRequest("City/Image/Add").
                AddFileContent(fileInfo, "text/plain").
                ExecuteAsync<Response>();
// POST text file at http://MyAPI.com/api/City/Add 
```


### Download file
```cs
string filePath = "c:\map.pdf";
FileInfo fileInfo = await client.
                GetRequest("City/map.pdf").
                DownloadFileAsync("c:\map.pdf");
// GET http://MyAPI.com/api/City/map.pdf 
```

## Get raw HttpResponseMessage

```cs

var response = await client.
                PostRequest("City/Add").
                AddFormParameter("country", "France").
                AddFormParameter("name", "Paris").
                ExecuteAsHttpResponseMessageAsync();
// POST http://MyAPI.com/api/City/Add with from url encoded content
```

## Get raw string result

```cs

var response = await client.
                PostRequest("City/Add").
                AddFormParameter("country", "France").
                AddFormParameter("name", "Paris").
                ExecuteAsStringAsync();
// POST http://MyAPI.com/api/City/Add with from url encoded content
```

## multi-part form data

```cs
// With 2 json content
var city1 = new City() { Name = "Paris" , Country = "France"};
var city2 = new City() { Name = "Ajaccio" , Country = "France"};
var response = await client.NewRequest(HttpVerb.Post, "City").
await client.PostRequest("MultiPart/Test").
              AsMultiPartFromDataRequest().
              AddContent<City>(city1, "city1", "city1.json").
              AddContent<City>(city2, "city2", "city2.json").
              ExecuteAsync();


// With 2 byte array content
byte[] byteArray1 = ...
byte[] byteArray2 = ...           
              
await client.PostRequest("MultiPart/Test").
              AsMultiPartFromDataRequest().
              AddByteArray(byteArray1, "request", "request2.bin").
              AddByteArray(byteArray2, "request", "request2.bin")
              ExecuteAsync();
  

// With 2 streams content        
Stream1 stream1 = ...
Stream stream2 = ...         
await client.PostRequest("MultiPart/Test").
              AsMultiPartFromDataRequest().
              AddStream(stream1, "request", "request2.bin").
              AddStream(stream2, "request", "request2.bin")
              ExecuteAsync();
              
              
// With 2 files content           

var fileInfo1 = new FileInfo("myTextFile1.txt");
var fileInfo2 = new FileInfo("myTextFile2.txt");

var response = await client.
                PostRequest("City/Image/Add").
                AsMultiPartFromDataRequest().
                AddFileContent(fileInfo1, "text/plain").
                AddFileContent(fileInfo2, "text/plain").
                ExecuteAsync<Response>();

// With mixed content                  
await client.PostRequest("MultiPart/Test").
              AsMultiPartFromDataRequest().
              AddContent<City>(city1, "city1", "city1.json").
              AddByteArray(byteArray1, "request", "request2.bin").
              AddStream(stream2, "request", "request2.bin")
              ExecuteAsync();
```


## Streams and bytes array
You can use as content : streams or byte arrays.
If you use these methods no serializer will be used.

### Streams
```cs

// Read stream response
 Stream stream = await client.
              GetRequest("File").
              WithStreamResponse().
              ExecuteAsStreamAsync();
// Post Stream as content
await client.PostRequest("File/Add").
            AddStreamContent(stream).
            ExecuteAsync();
```

### Bytes array
```cs
// Read byte array response         
byte[] byteArray = await client.
              GetRequest("File").
              ExecuteAsByteArrayAsync();

// Read byte array as content
await client.
            PostRequest("File/Add").
            AddByteArrayContent(byteArray).
            ExecuteAsync();
```


## Error handling
All requests can throw 3 exceptions : 

* ConnectionException : thrown when the request can't reach the server
* HttpException : thrown when the server has invalid error code
* DeserializeException : thrown when the deserializer can't deserialize the response

### Catch a specific error code
```cs
using Tiny.Http;
string cityName = "Paris";
try
{ 
   var response = await client.
     GetRequest("City").
     AddQueryParameter("Name", cityName).
     ExecuteAsync<City>();
}
catch (HttpException ex) when (ex.StatusCode == System.Net.HttpStatusCode.NotFound)
{
   throw new CityNotFoundException(cityName);
}
catch (HttpException ex) when (ex.StatusCode == System.Net.HttpStatusCode.InternalServerError)
{
   throw new ServerErrorException($"{ex.Message} {ex.ReasonPhrase}");
}
```

## Formatters 

By default : 
 * the Json is used as default Formatter.
 * Xml Formatter is added in Formatters

Each formatter have a list of supported media types.
It allow TinyHttpClient to detect which formatter will be used.
If the no formatter is found it use the default formatter.

### Add a new formatter
Add a new custom formatter as default formatter.
```cs
bool isDefaultFormatter = true;
var customFormatter = new CustomFormatter();
client.Add(customFormatter, isDefaultFormatter);
```

### Remove a formatter
```cs
var lastFormatter = client.Formatters.Where( f=> f is XmlSerializer>).First();
client.Remove(lastFormatter);
```

### Define a specific formatter for one request
```cs
IFormatter serializer = new XmlFormatter();
 var response = await client.
     PostRequest("City", city, serializer).
     ExecuteAsync();
```

### Define a specific formatter for one request
```cs
IFormatter deserializer = new XmlFormatter();

 var response = await client.
     GetRequest("City").
     AddQueryParameter("Name", cityName).
     ExecuteAsync<City>(deserializer);
```

### Custom formatter

You create your own serializers/deserializer by implementing IFormatter

For example the implementation of XmlFormatter is really simple : 
```cs
public class XmlFormatter : IFormatter
{

   public string DefaultMediaType => "application/xml";

   public IEnumerable<string> SupportedMediaTypes
   {
      get
      {
         yield return "application/xml";
         yield return "text/xml";
      }
   }

   public T Deserialize<T>(Stream stream, Encoding encoding)
   {
      using (var reader = new StreamReader(stream, encoding))
      {
         var serializer = new XmlSerializer(typeof(T));
         return (T)serializer.Deserialize(reader);
      }
   }

   public string Serialize<T>(T data, Encoding encoding)
   {
         if (data == default)
         {
             return null;
         }

         var serializer = new XmlSerializer(data.GetType());
         using (var stringWriter = new DynamicEncodingStringWriter(encoding))
         {
            serializer.Serialize(stringWriter, data);
            return stringWriter.ToString();
         }
      }
   }
```

## Logging events

```cs
using Tiny.Http;
var client = new TinyHttpClient("http://MyAPI.com/api");


client.SendingRequest += (sender , e) =>
{    
    Debug.WriteLine($"Sending RequestId = {e.RequestId}, Method = {e.Method}, Uri = {e.Uri}");
};

client.ReceivedResponse += (sender , e)=>
{
    Debug.WriteLine($"Received RequestId = {e.RequestId}, Method = {e.Method}, Uri = {e.Uri}, StatusCode = {e.StatusCode}, ElapsedTime = {ToReadableString(e.ElapsedTime)}");
};

client.FailedToGetResponse  += (sender , e)=>
{
    Debug.WriteLine($"FailedToGetResponse RequestId = {e.RequestId}, Method = {e.Method}, Uri = {e.Uri}, ElapsedTime = {ToReadableString(e.ElapsedTime)}");
}

```
