# strykerin - Update Dec 19th

As per my last update, 2 weeks ago my focus was on reading the existing tests on go-ethereum for discv4. More specifically I was reading how the `ping` and `pong` tests were done so I could learn the pattern.

Last week, I started to implement the `validation` on the `bond` function on `discv4test.go`. On the discovery protocol, in order to prevent traffic amplification attacks, nodes must 'bond' in order for them to start to exchange messages, this is called 'Endpoint Proof'.

On tests, when validating if we got correctly a `pong`, we are simply verifying the type from the message and if `pong`, then we continue the test. The plan for this week is to add the validation for the `pong` messages.

Here below is where I'll be adding the `pong` validation test:

```golang
// bond performs the endpoint proof with the remote node.
func bond(t *utesting.T, te *testenv) {
 ...
  switch req.(type) {
  case *v4wire.Ping:
   te.send(te.l1, &v4wire.Pong{
    To:         te.remoteEndpoint(),
    ReplyTok:   hash,
    Expiration: futureExpiration(),
   })
   gotPing = true
  case *v4wire.Pong:
   // TODO: maybe verify pong data here
   gotPong = true
  }
 }
}
```
