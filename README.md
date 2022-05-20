# mist

A (hopefully) nice, basic Gleam web server

## Installation

This package can be added to your Gleam project:

```sh
gleam add mist
```

and its documentation can be found at <https://hexdocs.pm/mist>.

## Usage

Right now there are 2 options.  Let's say you want a "simple" HTTP server that
you can customize to your heart's content.  In that case, you want:

```gleam
pub fn main() {
  assert Ok(_) = mist.serve(
    8080,
    http.handler(fn(_req) {
      response.new(200)
      // ...
    })
  )
  erlang.sleep_forever()
}
```

Maybe you also want to work with websockets.  Maybe those should only be
upgradable at a certain endpoint.  For that, you can use the router module.
For example:

```gleam
pub fn main() {
  let my_router =
    router.new([
      Http1(
        ["home"],
        fn(_req) {
          response.new(200)
          |> response.set_body(<<"sup home boy":utf8>>)
        },
      ),
      Websocket(["echo", "test"], websocket.echo_handler),
      Http1(
        ["*"],
        fn(_req) {
          response.new(200)
          |> response.set_body(<<"Hello, world!":utf8>>)
        },
      ),
    ])
  assert Ok(_) = glisten.serve(
    8080,
    my_router,
    router.new_state()
  )
  erlang.sleep_forever()
}
```

If you need something a little more complex, you can always use the helpers
exported by the various `glisten`/`mist` modules.

#### HTTP Hello World
```gleam
pub fn main() {
  assert Ok(_) = glisten.serve(
    8080,
    http.handler(fn(_req) {
      response.new(200)
      |> response.set_body(<<"hello, world!":utf8>>)
    }),
    Nil
  )
  erlang.sleep_forever()
}
```

#### Full HTTP echo handler
```gleam
pub fn main() {
  let service = fn(req: Request(BitString)) -> Response(BitString) {
    response.new(200)
    |> response.set_body(req.body)
  }
  assert Ok(_) = glisten.serve(
    8080,
    router.new([Http1(["*"], service)]),
    router.new_state(),
  )
  erlang.sleep_forever()
}
```

#### Websocket echo handler
```gleam
pub fn main() {
  assert Ok(_) = glisten.serve(
    8080,
    router.new([Websocket(["echo", "test"], websocket.echo_handler)]),
    router.new_state(),
  )
  erlang.sleep_forever()
}
```

## Notes

This is still very rough.  There are no tests, and as noted above you can't just
send where you implement it.

In some [not-very-scientific benchmarking](https://gist.github.com/rawhat/11ab57ef8dde4170304adc01c8c05a99), it seemed to do roughly as well as
ThousandIsland.  I am just using that as a reference point, certainly not trying
to draw any comparisons any time soon!
