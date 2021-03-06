#Ohmage API Python Client#

This is a thin Python client for the Ohmage server, a reference openmHealth DSU. It consists of a
single class, OhmageApi, which represents a connection to the server, and a few supporting
classes for representing Ohmage entities.

**NOTE:** The API is not yet complete; I'm implementing it in a top-down fashion (e.g. adding
functionality as the projects it supports require it). I anticipate that it'll be finished
soon, but in the meantime drop me a line if you need something that's missing.

## Connecting to Ohmage via OhmageApi

~~~python
from ohmagekit.clients import OhmageApi

# construct an api handle. note that no connection occurs until
# you make a request.
api = OhmageApi(<server>)

# OhmageApi allows you to either specify your credentials with each
# request, or use the convenience method login() to save them in
# the api handle. Any api method that requires credentials will
# use the cached credentials if explicit credentials are not supplied.
api.login(<username>, <password>)

# now the api handle contains both a short-use token and a hashed
# password which can be used indefinitely. if either become invalid,
# the api will throw an OhmageApiException containing a code 0200.
~~~
    
## Performing a simple request and checking for errors

~~~python
# query for a list of campaigns, which will be returned as a Python object.
# granted that this is a thin client, the response is parsed as JSON and
# returned verbatim. also, validating parameters is largely pushed to the server.
try:
    result = api.campaign_read(output_format="short")
except OhmageApi.OhmageApiException as ex:
    # let's do something special if there was an auth error (e.g. code 0200)
    if 200 in ex.codes():
        print "Your login credentials are invalid, exiting."
        exit(-1)
    # otherwise, pass it up to our caller
    raise

# note that other things may go wrong (you may lose your network connection,
# or the Ohmage server may be inaccessible, for instance). in this case, the
# api throws standard HTTPError exceptions, as detailed in the urllib2 documentation.

# practically all the ohmage api calls return a dictionary with two elements:
# - result['status'], which contains the status of the call, and
# - result['data'], which contains the actual data for which we queried.
# api calls that write data typically include only the 'status' key.

# let's loop through the campaign data as an exercise
for urn, campaign in result['data'].items():
    print "Campaign %s has URN %s" % (campaign.name, urn)

# there's no need to close the api handle, so we're done.
# perhaps i'll implement persistent connections in the future,
# which may necessitate closing the handle to free up resources.
~~~
