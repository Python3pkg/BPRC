# BPRC 
[![Build Status](https://travis-ci.org/bradwood/BPRC.svg?branch=master)](https://travis-ci.org/bradwood/BPRC)
[![license](https://img.shields.io/github/license/mashape/apistatus.svg?maxAge=2592000)](https://github.com/bradwood/BPRC/blob/master/LICENSE)

Batch Processing RESTFul Client

A DevOps tool to script calling a load of RESTFul JSON endpoints, with the ability to grab data from the JSON payload and use it in subsequent calls in the recipe.

## What it does
If you're a Dev/DevOps engineer you may have been faced with a situation where you find yourself writing a shell script to automate getting JSON data from one or more RESTful endpoints using `curl` or `wget` or `httpie`, and then parsing the output using `sed`, `grep` or `jq`. This tool is designed to provide a generic, simplied, yet powerful means of writing such scripts in the form of a simple _recipe_ specification, rather than a shell script. 

## To install
This is a Python application which is written in Python 3. It has only currently been tested on Linux (Ubuntu 14.04).

### Pre-requisites
Make sure you've installed Python 3 and Pip 3
```bash
apt-get install -y python3
apt-get install -y python3-pip
```

### Installation
Install like this:
```bash
pip3 install bprc
```

## How it works
The recipe is specified in a single YAML file which describes:
 - the list of URLs that need to be visited, in order
 - the HTTP method to use for each
 - the headers, querystring and body data to include for each step of the recipe

Additionally, the YAML recipe file supports the ability to  grab data from any part of any of the HTTP requests or responses in earlier steps in the recipe and insert them into later steps using a PHP-like construct. For example, say I have a 10-step recipe specified and in step 7 I need to POST some data that I received in step 3's reponse. I can include a construct like this in any part of the YAML file: 
```
<%=steps[3].response.body["id"]%>
```
Assuming that step 3 did indeed contain a parameter called `id` in it's JSON response payload, this data will then be substituted in the specified part of step 10's request. 

### Sample Recipe
This functionality is best illustrated with a complete recipe file as shown below.
```yaml
--- #sample recipe
recipe:
  -  # step0
    name: HTTPBIN call
    httpmethod: GET
    URL: http://httpbin.org/get
    request:
      body:
        name: json name parameter
        url: http://wiremock/blah
        booleanflag: false
      headers:
        X-ABC: 1231233425435fsdf
      querystring:
        keya: vala
        keyb: valb
    response: # set up this response section if you need to pull out data here for use later in the recipe.
              # the YAML hierarchy will map to the JSON response obtained and the the values received from
              # the call will be inserted into the appropriate response variables so that subsequent steps
              # can access them with the php-like construct
      body:
        id: this_is_a_param
        origin: somval
      headers:
        Authorization:
      code: #HTTP response code
  -
    name: Load OAUTH Plugin
    httpmethod: POST
    URL: http://httpbin.org/post
    request:
      querystring:
        key3: value3
      body:
        key4: valueprefix <%=steps[0].request.headers["X-ABC"]%>
      headers:
        blah: blahbha
        Authorisation: bearer <%=steps[0].request.querystring["keyb"]%>
   
```
## Other features
`bprc` provides the following features:
 - robust logging support
 - saving output files as raw HTTP or JSON
 - SSL support (including the ability to ingore invalid server certificates)
 - Dry-run *(Not implemented yet)*
 - verbose and/or debug output

## Known issues
The following are known areas for improvement:
- poor tolerance of badly formatted YAML
- `--dry-run` option not implemented
- poor test coverage and test automation

## Planned improvements
- The ability to include a file's data into the recipe using an `@` prefix
- Fixing the known issues above
