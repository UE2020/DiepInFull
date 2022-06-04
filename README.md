# DiepInFull
Protocol information regarding the browser game Diep.io. Includes everything, unlike ABCxff's DiepInDepth (DID).

## Protocol

Diep.io's protocol is a bizarre over complication of something simple. For example, you can decode then parse incoming packets through a HTTPS WebSocket connection over a TCP stream - or, alternatively, you can open https://diep.io/". The protocol is largely unknown to me at the moment. I haven't looked at anything besides the clientbound 0x00 packet, which contains all entity information.





Packets exist and they scare me.

### Inbound 0x00

The inbound 0x00 begins with a 1 byte packet header as all packets do:

```js
let typ = buf.get_u8();
```

After the packet header comes the "game tick" encoded as a varuint. The game tick is the amount of server-side ticks have passed since the arena came online. As a security mechanism (security-by-obscurity), the game tick must be XOR'd by 4061406 as of the latest build.

```js
let gameTick = buf.get_vu() ^ 4061406;
```

0x00 packets update the game state using 3 mechanisms: updates, deletions, and creations. After the packet header is a varuint which represents the amount of deletions contained in that packet. Just like the game tick, the delete count must also be XOR'd.

```js
let deleteCount = buf.get_vu() ^ ((game_tick + 80) & 127); // dw about complexity
```

Naturally, a list of all the deletions comes after the delete count. Entity IDs in Diep.io are represented as two varuints: a hash and an ID. Reading all the deletions is simple:

```js
for (let i = 0; i < deleteCount; i++) {
  console.log(' Hash:', buf.get_vu());
  console.log(' ID:', buf.get_vu());
}
```
