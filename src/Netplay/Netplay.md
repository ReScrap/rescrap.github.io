# Encryption  (v1.1)

Fixed Key: `020406080a0c0e10121416181a1c1e20222426282a2c2e30323436383a3c3e40`
Encrypted Packet Structure:
```rust
struct Packet {
	#[len=pad(nonce_len,16)]
	nonce: Vec<u8>,
	#[len=pad(data_len,16)]
	data: Vec<u8>,
	nonce_len: u64,
	data_len: u64
	tag: [u8;16]
}
```
```
      00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
0000  ED C9 C2 F4 7C 6D F2 54 42 EF 46 F6 00 00 00 00  ....|m.TB.F.....

0010  68 13 5C 9A 2B 18 DB 9C 76 BE A0 8A 3E 49 79 3C  h.\.+...v...>Iy<
0020  8D 7A C4 4C 8B B0 A4 94 E5 B5 89 54 A6 ED 6D 75  .z.L.......T..mu
0030  1A CA A8 4B 22 B5 03 84 F7 3C DE 4E B0 30 81 29  ...K"....<.N.0.)
0040  3B 70 45 15 33 C0 97 67 85 6B 28 EF 2E 2E D1 83  ;pE.3..g.k(.....
0050  E6 56 A7 81 53 89 3E 52 D8 82 CF 77 92 CF C2 D6  .V..S.>R...w....
0060  9F 37 C5 DE EE 14 4D 3F 1F 82 32 7E 00 00 00 00  .7....M?..2~....

0070  0C 00 00 00 00 00 00 00 5C 00 00 00 00 00 00 00  ........\.......

0080  89 7A A8 32 93 56 B6 68 24 E0 58 63 7F 70 5A D2  .z.2.V.h$.Xc.pZ.
```
Decryption algorithm (Pseudocode):
```python
cipher = ChaCha20(key,pkt.nonce)
packet_key = cipher.decrypt(key)
cipher.seek(packet_key.len()+32)
data = cipher.decrypt(pkt.data)
```
Decrypted:
```
      00 01 02 03 04 05 06 07 08 09 0A 0B 0C 0D 0E 0F
0000  BA CE 01 01 8B 6D 06 00 00 00 01 4D 53 45 31 3A  .....m.....MSE1:
0010  20 46 6C 61 67 20 48 75 6E 74 20 32 2D 36 00 05   Flag Hunt 2-6..
0020  00 00 00 00 00 05 18 01 88 AA 9B 46 6C 61 67 48  ...........FlagH
0030  75 6E 74 00 54 A9 2F 05 F0 B4 4F 43 41 00 00 00  unt.T./...OCA...
0040  4D 07 18 01 60 A3 00 05 00 00 80 3F 20 4A DA 3D  M...`......? J.=
0050  68 F8 8F 01 19 6A 71 77 20 8B FB 76              h....jqw ..v
```

# Netplay

Game Info Packet

```
Server 'B':FZ (0/10) Ver 1.0 at 192.168.99.1:28086
[0-2] ID (0xbace)
[2-4] Version
[4-5] port (16-bit)
[6-7] max_players (16-bit)
[8-9] curr_player (16-bit)
[10-x] server name (char*)

           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
0019fdc0  ba ce 00 01 b6 6d 0a 00 00 00 42 00 30 fe 19 00  .....m....B.0...
0019fdd0  ff ff ff ff 27 2b b3 9b c7 3e bb 00 9c af 29 00  ....'+...>....).
0019fde0  db 69 00 00 00 00 00 00 00 00 44 65 61 74 68 4d  .i........DeathM
0019fdf0  61 74 63 68 00 00 00 00 ff ff 46 5a 00 4a 91 f0  atch......FZ.J..
0019fe00  92 8b 57 4e 7f 00 00 00 10 21 fe 38 0d ae 00 00  ..WN.....!.8....
0019fe10  f0 ce f3 36 a0 e8 0b 77 a0 e8                    ...6...w..
```


Player Join Packet

```
[0-3] header/ID?
[6-x] Player name

           0  1  2  3  4  5  6  7  8  9  A  B  C  D  E  F  0123456789ABCDEF
09c9dfe8  7f 47 00 00 00 0e 55 6e 6e 61 6d 65 64 20 50 6c  .G....Unnamed Pl
09c9dff8  61 79 65 72 06 53 42 6f 73 73 31 b9 00 07 50 5f  ayer.SBoss1...P_
09c9e008  42 65 74 74 79 06 4d 42 4f 53 53 31 06 4d 42 4f  Betty.MBOSS1.MBO
09c9e018  53 53 31 00 00 10 30 2c 31 35 2c 30 2c 30 2c 31  SS1...0,15,0,0,1
09c9e028  35 2c 31 35 2c 31 02 00 00 00                    5,15,1....
```

 Message                                  | Description                                                       
 ---------------------------------------- | ----------------------------------------------------------------- 
 `5c68625c32383230395c73637261706c616e64` | "Scrapland Server" announcement broadcast (`\hb\28209\scrapland`) 
 `7f01000007`                             | Retrieve Game info                                                
 `48423d35323932322c3235363a323830383600` | Connection Information (`HB=52922,256:28086`)                     
