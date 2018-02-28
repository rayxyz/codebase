> iota
```go
const (
	_ = iota
	// CONNECT : Connect
	CONNECT
	// CONNACK : Connect acknowledgement
	CONNACK
	// PUBLISH : Publish
	PUBLISH
	// PUBACK : Publish acknowledgement
	PUBACK
	// PUBREC : Publish receive
	PUBREC
	// PUBREL : Publish release
	PUBREL
	// PUBCOMP : Publish complete
	PUBCOMP
	// SUBSCRIBE : Subscribe
	SUBSCRIBE
	// SUBACK : Subscribe acknowledgement
	SUBACK
	// UNSUBSCRIBE : Unsubscribe
	UNSUBSCRIBE
	// UNSUBACK : Unsubscribe acknowledgement
	UNSUBACK
	// PINGREQ : Ping request
	PINGREQ
	// PINGRESP : Ping response
	PINGRESP
	// DISCONNECT : Disconnect
	DISCONNECT
)
```

> checksum
```go
import (
	"fmt"
	"testing"
)

func checksum(b []byte) uint16 {
	csumcv := len(b) - 1 // checksum coverage
	s := uint16(0)
	for i := 0; i < csumcv; i += 2 {
		// s += uint32(b[i+1])<<8 | uint32(b[i])
		s += uint16(b[i])
		s += s & uint16(b[i+1])
	}
	// if csumcv&1 == 0 {
	// 	s += uint32(b[csumcv])
	// }
	// s = s>>16 + s&0xffff
	// s = s + s>>16

	return ^s
}

func TestChecksum(t *testing.T) {
	i := checksum([]byte{123, 2, 5, 6, 9, 10})
	fmt.Println("checksum => ", i)
	fmt.Println("reverse checksum => ", ^i)
	fmt.Printf("checksum bits => %08b\n", i)
	fmt.Printf("reverse checksum bits => %08b\n", ^i)
	flags := 0x00
	flags |= 0xff
	flags &= 0xfe
	fmt.Printf("flags %08b\n", flags)
	iflag := 122
	fmt.Printf("iflag => %08b\n", iflag)
	fmt.Printf("reverse iflag => %08b\n", ^iflag)
}
```

> bitwise
```go
func TestBits(t *testing.T) {
	// x := byte(0xE9)
	x := byte('a')
	fmt.Printf("%b\n", x)
	fmt.Println("Bits, least to most significant:")
	for i := uint(0); i < 8; i++ {
		fmt.Println(x & (1 << i) >> i)
	}
}

func TestBits02(t *testing.T) {
	var a uint = 3
	fmt.Printf("%08b\n", a)
	fmt.Printf("%08b\n", a<<1)
	fmt.Printf("%08b\n", a<<2)
	fmt.Printf("%032b\n", a<<3)
	x32bitStr := []byte(fmt.Sprintf("%032b\n", a<<6))
	fmt.Println("32 bits of a interger => ", x32bitStr)
	fmt.Println("bits.Len(x) => ", bits.Len(a<<3))
	fmt.Println(byte(4<<4 | (20 >> 2 & 0x0f)))
}

func TestBinary(t *testing.T) {
	ba := []byte{1, 2}
	// 2 bytes, 16 bits
	// 00000001 00000010
	num := binary.BigEndian.Uint16(ba[0:])
	fmt.Println("num => ", num, "num == 258 : ", (num == 258))
	fmt.Printf("num in bits => %016b\n", num)

	ba = []byte{1, 2, 3, 4}
	numx := binary.BigEndian.Uint32(ba)
	fmt.Println("numx in decimal => ", numx)
	fmt.Printf("numx in bits => %032b\n", numx)

	bx := make([]byte, 4)
	binary.BigEndian.PutUint16(bx[0:2], uint16(65530))
	fmt.Println("bx[0] => ", bx[0], " bx[1] => ", bx[1])
	fmt.Printf("bx[0:2] bits => %08b\n", bx[0:2])
	bitsStr := fmt.Sprintf("%08b", bx)
	fmt.Println("bitsstring => ", bitsStr)
}

func TestBitSetunset(t *testing.T) {
	b := 7 // bits : 0000 0111
	fmt.Printf("bits of 7 => %08b decimal => %d\n", b, b)
	b |= 1 << 4
	fmt.Printf("after set pos 4 to 1 => %08b decimal => %d\n", b, b)
	fmt.Println("the bit at pos 4 is => ", (b&(1<<4) != 0))
	b &^= 1 << 4
	fmt.Printf("after unset pos 4 to 0 => %08b decimal => %d\n", b, b)
	fmt.Println("the bit at pos 2 is => ", (b&(1<<2) != 0))
	fmt.Println("the bit at pos 3 is => ", (b&(1<<4) != 0))
	fmt.Printf("(1<<3 | 1<<1) => %08b\n", (1<<3 | 1<<1))
}
```
