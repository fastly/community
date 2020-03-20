Why have your user go ALLLLLLL the way to your origin just to be served back a JSON response? Great question, I dont have a great answer either. What I do have is a way to serve JSON responses from the edge, using Fastly, VCL, and some interesting syntax.

The basic idea is that you cannot just create a simple `vcl_error` synthetic response like you would with HTML. VCL and JSON are not the best of friends but will play nicely if you can get them to talk together.

For a quick review a typical HTML response looks like:

```
if (obj.status == <some custom status>) {
  set obj.status = <some HTML response code>;
  set obj.response = “<Some message>”;
  synthetic {"
    <html>
      <head>
      </head>
      <body>
        <h1>Custom Response</h1>
      </body>
    </html>
  "};
  return(deliver);
}

```

Pretty easy. However, if you tried to hack together a solution for JSON by simply changing the HTML document, you’d hit a few road bumps. First thing to do is to tell the requesting client what kind of document it is receiving. That’s where `set obj.http.Content-Type = "application/json”` comes in. Then in the `synthetic potion we need to have `synthetic {json”{<your JSON response>}"json}`. This way VCL knows what, exactly, we are sending out and how to handle it. Then simply ` return(deliver);` and the requesting client is getting back well formed, fast JSON responses. To give you a quick idea of what it can look like is this ode sample below, along with a Fiddle (https://fiddle.fastlydemo.net/fiddle/7abef350).

```
if (obj.status == <some custom status>) {
  set obj.status = 200;
  set obj.http.Content-Type = "application/json";
  synthetic {json"{"Key1": "Value1", "Key2": "Value2"}"json};
  return(deliver);
}
```

One other thing that you may want to do is have a lookup tables that holds a set JSON responses. This may be useful if you know you have different contact info for your support teams and want to be able to send that info from the requesting client without holding that data on our origin servers. The below fiddle will give you. Quick idea on how to incorporate lookup tables with synthetic JSON responses.

https://fiddle.fastlydemo.net/fiddle/ac2f3c67

As always, if you have questions or need assistance, feel free to email support at support@fastly.com and out support engineers will be happy to assist you.
