# route-by-xpath
Custom Plugin in python : Route dynamically a request to an upstream if a xpath condition is validated

# Kong plugin: XML Request Handling
This Kong plugin is developed in Python and uses the Python library [lxml.etree](https://pypi.org/project/lxml/)  binding for the GNOME C libraries [libxml2](https://gitlab.gnome.org/GNOME/libxml2#libxml2) and [libxslt](https://gitlab.gnome.org/GNOME/libxslt#libxslt).

The plugin handles the XML Request (in the HTTP body) in this order:
1) Read a xpath value
2) Compare this vlaue with a condition
3) Update upstream target to route dynamically a request

## How to test route-by-xpath with ```calcWebService/Calc.asmx```
### Create a Kong Service and route
1) Create a Kong Service named ```calcWebService``` with this URL: https://ecs.syr.edu/faculty/fawcett/Handouts/cse775/code/calcWebService/Calc.asmx.
This simple backend Web Service adds 2 numbers.

2) Create a Route on the Service ```calcWebService``` with the ```path``` value ```/calcWebService```

3) Call the ```calcWebService``` through the Kong Gateway Route by using [httpie](https://httpie.io/) tool
```
http POST http://localhost:8000/calcWebService \
Content-Type:"text/xml; charset=utf-8" \
SOAPAction: "http://tempuri.org/Add" \
--raw "<?xml version=\"1.0\" encoding=\"utf-8\"?>
<soap:Envelope xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xmlns:xsd=\"http://www.w3.org/2001/XMLSchema\"
xmlns:soap=\"http://schemas.xmlsoap.org/soap/envelope/\">
  <soap:Body>
    <Add xmlns=\"http://tempuri.org/\">
      <a>5</a>
      <b>7</b>
    </Add>
  </soap:Body>
</soap:Envelope>"
```

The expected result is ```12```:
```xml
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/" ...>
  <soap:Body>
    <AddResponse xmlns="http://tempuri.org/">
      <AddResult>12</AddResult>
    </AddResponse>
  </soap:Body>
</soap:Envelope>
```

4) Install the ```route-by-xpath``` plugin on the Service ```calcWebService``` and set parameters. 

Configure the plugin with:
- ```XPath``` xpath to use for the condition. ex : .//{http://tempuri.org/}a,
- ```Condition``` value expected. ex : 5
- ```Upstream``` name of an upstream created/existing in Kong (string of the upstream name)
- ```path``` path if you want to overide it

If value of tag "a" equals 5, the plugin will route the request to the upstream

