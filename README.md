# cowboy_revproxy

cowboy_revproxy is a simple TCP routing proxy (layer 7) in erlang and
using Cowboy connection handling. It lets you configure the routine
logic in Erlang. 

If you need to proxy connections to different backend servers depending
on the contents of the transmission, then cowboy_revproxy will helps
you. It's heavily inspidered from [proxymachine](https://github.com/mojombo/proxymachine).


## Build

You need [rebar](http://github.com/basho/rebar) to build
cowboy_revproxy:

    $ rebar get-deps
    $ rebar compile 

## Usage

The idea is simple, once a request is coming to the proxy the data is
passed to a proxy function until this function return a remote
connection or tell to the proxy to close the connection. 

**Valid returns values** are :

* `stop` -> close the connection
* `{stop, Reply}` -> send Reply to the client and close the connection
* `{remote, Remote}` -> return the address of the remote connection to
  proxy. Remote can be one of the following:
    - `{ip, port}`
    - `{ssl, Ip, Port, Options}`, where options are ssl options
      (keyfile, certfile & password).
* `[{remote, Remote}, {data, Data}]` -> same as above, but data passed to
  the proxy function is replaced by `Data` and will be send to the remote
  connection.
* `[{remote, Remote}, {data, Data}, {reply, Reply}]` -> same as above
  but reply `Reply` to the client.

## Example

Here is a simple example of function proxying the port 8080 to google.com:

    -module(cowboy_revproxy_example).
    -export([start/0, proxy/1]).

    proxy(_Data) ->
        {remote, {"www.google.com", 80}}.

    start() ->
        application:start(cowboy),
        application:start(cowboy_revproxy),
            cowboy:start_listener(http, 100,
            cowboy_tcp_transport, [{port, 8080}],
            cowboy_revproxy, [{proxy, {?MODULE, proxy}}]).


To test it do, in the source folder:

    $ rebar get-deps compile
    $ erl -pa ebin -pa deps/cowboy/ebin
      
Then in the erlang shell:

    1> cowboy_revproxy_example:start().

and go on http://127.0.0.1:8080/ , it should display the google page.
