# ubirch-protocol for python

This is an implementation of the [ubirch-protocol](https://github.com/ubirch/ubirch-protocol)
for [Python 3](https://www.python.org/). Please see [ubirch-protocol](https://github.com/ubirch/ubirch-protocol)
for details.

## Usage

### Creating keypair and messages

```python
import ubirch
from ubirch.ubirch_protocol import CHAINED
from uuid import UUID
import binascii

# create a keystore for the device keypair
keystore = ubirch.KeyStore("device.jks", "keystore")

# create a UUID that identifies the device and load or create a keypair
uuid = UUID(hex="575A5601FD744F8EB6AEEF592CDEE12C")
if not keystore.exists_signing_key(uuid):
    keystore.create_ed25519_keypair(uuid)

# implement the _sign method on the ubirch.Protocol to use the just created
# keys to sign the message and add methods to save and load the last signature
class ProtocolImpl(ubirch.Protocol):
    def _sign(self, message: bytes) -> bytes:
        return keystore.find_signing_key(uuid).sign(message)

    def _save_signature(self, signature: bytes) -> None:
        try:
            with open(uuid.hex + ".sig", "wb+") as f:
                f.write(signature)
        except Exception as e:
            print("can't write signature file: {}".format(e))

    def _load_signature(self) -> bytes:
        try:
            with open(uuid.hex + ".sig", "rb") as f:
                return f.read(64)
        except Exception as e:
            print("can't read signature file: {}".format(e))
        return b'\0' * 64


proto = ProtocolImpl(CHAINED)
print(binascii.hexlify(proto.message(uuid.bytes, 0x00, [1, 2, 3])))
print(binascii.hexlify(proto.message(uuid.bytes, 0x00, [4, 5, 6])))
```
 
### Sending messages using the ubirch API

```python
# TODO
```

# License 

The protocol and its implementation are publicized under the [Apache License 2.0](LICENSE).