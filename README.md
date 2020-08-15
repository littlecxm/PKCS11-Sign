# Sign certificate in PKCS#11 with OpenSC & OpenSSL using Safenet in Windows

## Load Token Info

`pkcs11-tool --module=C:\\Windows\\system32\\etpkcs11.dll -L`

```cmd
Available slots:
Slot 0 (0x0): AKS ifdh 0
  (empty)
Slot 1 (0x1): AKS ifdh 1
  (empty)
Slot 2 (0x2): Rainbow Technologies iKeyVirtualReader 0
  token label        : CXM
  token manufacturer : ㏑ainbow Tech.
  token model        : Model 330
  token flags        : login required, rng, token initialized, PIN initialized, other flags=0x200
  hardware version   : 0.6
  firmware version   : 0.6
  serial num         : (***secret***)
  pin min/max        : 6/16
Slot 3 (0x3): Rainbow Technologies iKeyVirtualReader 1
...
```

## Check Token Info

`pkcs11-tool --module=C:\\Windows\\system32\\etpkcs11.dll --list-objects`

```cmd
Using slot 2 with a present token (0x2)
Public Key Object; RSA 2048 bits
  label:
  ID:         (***secret***)
  Usage:      encrypt, verify, wrap
  Access:     none
Certificate Object; type = X.509 cert
  label:      (***secret***)
  subject:    DN: CN=CXM CA, O=CXM
  ID:         (***secret***)
```

remember this `Public Key Object; RSA 2048 bits` ID, need fill in command after.

## Load Module

Using [OpenSC libp11 wrapper](https://github.com/OpenSC/libp11)

_you need using MSVC/MYSY2 to build it on windows_

Wrapper Path: `E:\git\libp11\src\pkcs11.dll`

Safenet Driver Path: `C:\\Windows\\system32\\etpkcs11.dll`

```cmd
engine dynamic -pre SO_PATH:E:\git\libp11\src\pkcs11.dll -pre ID:pkcs11 -pre NO_VCHECK:1 -pre LIST_ADD:1 -pre LOAD -pre "MODULE_PATH:C:\\Windows\\system32\\etpkcs11.dll" -pre VERBOSE
```

## Self-signed

Fill the `slot_(X)` and the `id_(x)` in `pkcs11-tool` shown before like `slot_2-id_aabbccddeeff001122`

```cmd
x509 -engine pkcs11 -CAkeyform engine -CAkey slot_5-id_2 -sha256 -CA ca.crt -CAcreateserial -req -in server.csr -out sb.crt
```

- -engine `pkcs11`

  using pkcs11 engine

- -CAkeyform `engine`

  follow engine (?)

- -CAkey `slot_2-id_aabbccddeeff001122`

  ca private key slot& id

- -CA `ca.crt`

  ca public cert

- -CAcreateserial

  CA serial number file is created if it does not exist

- -req -in `server.csr`
  client request

- -out `sb.crt`
  client output cert

# Issue

Fuck windows cannot recognized `Yubikey` Slot by OpenSC `libp11` openssl wrapper

SafeNet device can work so fucking fine

Tested with iKey 2032
