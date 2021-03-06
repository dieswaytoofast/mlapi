MercadoLibre API Client for Erlang
==================================

[MercadoLibre](http://www.mercadolibre.com/) is the biggest e-commerce site in
Latin America (similar to eBay) and it has recently released a REST API to
retrieve information from the published items, the users and most of the actions
that can be performed in the site. The API returns its results encoded in
[JSON](http://www.json.org/) and has a public interface that is freely
accessible and a private interface that can only be accessed by a MercadoLibre
user using [OAuth](http://oauth.net/) authentication in an application.

This implementation addresses the public and private sides of the API. There are
two main modules:
1) ``mlapi``: which provides uncached access to the API; and 2) ``mlapi_cache``:
which caches the results returned by the API. The cached entries are automatically
removed once they expire.

Requirements
============
You will need a fairly recent version of [Erlang](http://www.erlang.org/) and
[rebar](https://github.com/basho/rebar) installed in your path. So far it has
been tested with Erlang/OTP R14 and R15 on Ubuntu Linux 11.04, 11.10 and 12.04
but will most probably work on other platforms too.

Installation
============
Clone the [mlapi](https://github.com/jcomellas/mlapi) repository and issue the
following commands:

    git clone https://github.com/jcomellas/mlapi.git
    cd mlapi
    make depends
    make

That will download all the required Erlang dependencies and compile the project.
After that you can start using the modules in it.

Usage
=====
You can easily test the modules within the Erlang shell. To enter the shell with
the required paths already set run:

    make console

Once you're in the Erlang shell you need to start the ``mlapi`` application. You
can start it and its dependencies by doing:

    mlapi:start().

Now we're ready to rock. Keep in mind the following type specifications:

    -type error()             :: {error, Reason :: atom() | {atom(), any()}}.
    -type ejson_key()         :: binary().
    -type ejson_value()       :: binary() | boolean() | integer() | float() | null.
    -type ejson()             :: {[{ejson_key(), ejson_value() | ejson()}]}.
    -type proplist()          :: [{Key :: atom(), Value :: term()}].
    -type format()            :: ejson | proplist | record | dict | orddict | raw.
    -type option()            :: {format, format()} | {record, RecordName :: atom()} | refresh.
    -type response_element()  :: ejson() | proplist() | tuple() | dict() | orddict:orddict() | binary().
    -type response()          :: response_element() | [response_element()] | error().

All of the available functions that retrieve information from [MLAPI](http://www.mercadolibre.io/)
are very similar and follow a syntax like the following one:

    -spec mlapi:user(mlapi_user_id(), [mlapi:option()]) -> mlapi:response().

This is also a short version like:

    -spec mlapi:user(mlapi_user_id()) -> mlapi:response().

All the functions can receive options in the last argument. The most important
one would be the one to specify the format of the result. It follows the syntax:

    {format, Format :: mlapi:format()}

where ``Format`` can be one of:

<table>
 <thead>
  <tr><td>Format</td><td>Description</td></tr>
 </thead>
 <tbody>
  <tr><td>ejson</td><td>returns the JSON document as decoded by the ejson Erlang library (see https://github.com/benoitc/ejson)</td></tr>
  <tr><td>proplist</td><td>returns the parsed JSON document as a property list (see http://www.erlang.org/doc/man/proplists.html)</td></tr>
  <tr><td>record</td><td>returns the parsed JSON document as the corresponding record as defined in the mlapi.hrl header file</td></tr>
  <tr><td>dict</td><td>returns the parsed JSON document as an dict (see http://www.erlang.org/doc/man/dict.html)</td></tr>
  <tr><td>orddict</td><td>returns the parsed JSON document as an orddict (see http://www.erlang.org/doc/man/orddict.html)</td></tr>
  <tr><td>raw</td><td>returns the unparsed binary with the JSON document</td></tr>
 </tbody>
</table>

For example, if we wanted to format the result as a proplist we'd do:

    mlapi:sites([{format, proplist}]).

And we'd receive:

    [[{id,<<"MLA">>},{name,<<"Argentina">>}],
     [{id,<<"MLB">>},{name,<<"Brasil">>}],
     [{id,<<"MCO">>},{name,<<"Colombia">>}],
     [{id,<<"MCR">>},{name,<<"Costa Rica">>}],
     [{id,<<"MEC">>},{name,<<"Ecuador">>}],
     [{id,<<"MLC">>},{name,<<"Chile">>}],
     [{id,<<"MLM">>},{name,<<"Mexico">>}],
     [{id,<<"MLU">>},{name,<<"Uruguay">>}],
     [{id,<<"MLV">>},{name,<<"Venezuela">>}],
     [{id,<<"MPA">>},{name,<<"Panamá">>}],
     [{id,<<"MPE">>},{name,<<"Perú">>}],
     [{id,<<"MPT">>},{name,<<"Portugal">>}],
     [{id,<<"MRD">>},{name,<<"Dominicana">>}]]

Available Functionality
=======================
For the time being, you can retrieve the following information provided by [MLAPI](http://www.mercadolibre.io/):

- Search by keywords, by category, by seller ID and by seller nickname
- Sales
- Orders
- Items
- Users
- Categories
- Domains
- Pictures
- Global and category trends
- Geolocation information
- Credit card issuers
- Payment types and methods
- Listing exposures, types and their prices
- Currencies and their conversion rates
- Cities, states and countries
- Sites

You can check the exported functions in ``src/mlapi.erl`` to see the complete interface.

Cached Interface
================
There is a variant of the ``mlapi`` module called ``mlapi_cache`` that caches
the results it receives in Mnesia. The time-to-live of each type of result can
be specified in the ``mlapi_metatable`` Mnesia table (see ``src/mlapi_cache.erl``
for its definition). The interface is the same as the one provided by the
``mlapi`` module.

Accessing the Documents
=======================
The resulting documents can be accessed very easily with the use of normal
Erlang tools. In particular, I'd recommend Bob Ippolito's [kvc](https://github.com/etrepum/kvc)
for the proplist format and Anton Lavrik's [erlson](https://github.com/alavrik/erlson.git)
for the orddict format.
