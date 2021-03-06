# ElixirSense

An API/Server for Elixir projects that provides context-aware information for code completion, documentation, go/jump to definition, signature info and more.

## Usage

```
git clone https://github.com/elixir-lsp/elixir_sense.git
cd elixir_sense
elixir run.exs socket_type port env
```

Where:
- `socket_type` - Can be either `unix` (Unix domain socket) or `tcpip`.
- `port` - Specifies which port number to use. Setting port to `0` will make the underlying OS assign an available port number.
- `env` - The environment. Possible values are `dev` or `test`

Example (using Unix domain socket):

```
$ elixir run.exs unix 0 dev
ok:localhost:/tmp/elixir-sense-some_generated_unique_id.sock
```

Example (using TCP/IP):

```
$ elixir run.exs tcpip 0 dev
ok:localhost:56789:AUTH_TOKEN
```

> Note: AUTH_TOKEN is an authentication token generated by the server. All requests sent over tcp/ip must contain this token.

## Connecting to the server

The TCP server sends/receives data using a simple binary protocol. All messages are serialized into Erlang's [External Term Format](http://erlang.org/doc/apps/erts/erl_ext_dist.html). Clients that want to communicate with the server must serialize/deserialize data into/from this format.

### Example using :gen_tcp

```elixir
{:ok, socket} = :gen_tcp.connect({:local, '/tmp/elixir-sense-some_generated_unique_id.sock'}, 0, [:binary, active: false, packet: 4])

ElixirSense.Server.ContextLoader.set_context("dev", "PATH_TO_YOUR_PROJECT")

code = """
defmodule MyModule do
  alias List, as: MyList
  MyList.flatten(par0,
end
"""

request = %{
  "request_id" => 1,
  "auth_token" => nil,
  "request" => "signature",
  "payload" => %{
    "buffer" => code,
    "line" => 3,
    "column" => 23
  }
}

data = :erlang.term_to_binary(request)
:ok = :gen_tcp.send(socket, data)
{:ok, response} = :gen_tcp.recv(socket, 0)
:erlang.binary_to_term(response)

```

The output:

```elixir
%{request_id: 1,
  payload: %{
    active_param: 1,
    pipe_before: false,
    signatures: [
      %{documentation: "Flattens the given `list` of nested lists.",
        name: "flatten",
        params: ["list"],
        spec: "@spec flatten(deep_list) :: list when deep_list: [any | deep_list]"
      },
      %{documentation: "Flattens the given `list` of nested lists.\nThe list `tail` will be added at the end of\nthe flattened list.",
        name: "flatten",
        params: ["list", "tail"],
        spec: "@spec flatten(deep_list, [elem]) :: [elem] when deep_list: [elem | deep_list], elem: var"
      }
    ]},
  error: nil
}
```

### Example using `elixir-sense-client.js`

```javascript
let client = new ElixirSenseClient('localhost', '/tmp/elixir-sense-some_generated_unique_id.sock', null, "dev", PATH_TO_YOUR_PROJECT)

code = `
  defmodule MyModule do
    alias List, as: MyList
    MyList.flatten(par0,
  end
`;

client.send("signature", { buffer: code, line: 4, column: 25 }, (result) => {
  console.log(result);
});
```

## Testing

```
$ mix deps.get
$ mix test
```

A few of the tests require a source installation of Elixir which you can accomplish with [asdf](https://github.com/asdf-vm/asdf-elixir) (use `ref:v1.7.1`) or [kiex](https://github.com/taylor/kiex)

To run the tests that require a source installation of Elixir run:
```
mix test --include requires_source
```

For coverage:

```
mix coveralls
```

## Credits

- This project probably wouldn't even exist without all the work done by Samuel Tonini and all contributors from [alchemist-server](https://github.com/tonini/alchemist-server).
- The Expand feature was inspired by the [mex](https://github.com/mrluc/mex) tool by Luc Fueston. There's also a very nice post where he describes the whole process of [Building A Macro-Expansion Helper for IEx](http://blog.maketogether.com/building-a-macro-expansion-helper/).

## License (The MIT License)

Copyright (c) 2017 Marlus Saraiva

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
