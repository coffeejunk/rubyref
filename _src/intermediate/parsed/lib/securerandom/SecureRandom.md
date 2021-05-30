# SecureRandom

## Secure random number generator interface.

This library is an interface to secure random number generators which are
suitable for generating session keys in HTTP cookies, etc.

You can use this library in your application by requiring it:

    require 'securerandom'

It supports the following secure random number generators:

*   openssl
*   /dev/urandom
*   Win32


SecureRandom is extended by the Random::Formatter module which defines the
following methods:

*   alphanumeric
*   base64
*   choose
*   gen_random
*   hex
*   rand
*   random_bytes
*   random_number
*   urlsafe_base64
*   uuid


These methods are usable as class methods of SecureRandom such as
`SecureRandom.hex`.

### Examples

Generate random hexadecimal strings:

    require 'securerandom'

    SecureRandom.hex(10) #=> "52750b30ffbc7de3b362"
    SecureRandom.hex(10) #=> "92b15d6c8dc4beb5f559"
    SecureRandom.hex(13) #=> "39b290146bea6ce975c37cfc23"

Generate random base64 strings:

    SecureRandom.base64(10) #=> "EcmTPZwWRAozdA=="
    SecureRandom.base64(10) #=> "KO1nIU+p9DKxGg=="
    SecureRandom.base64(12) #=> "7kJSM/MzBJI+75j8"

Generate random binary strings:

    SecureRandom.random_bytes(10) #=> "\016\t{\370g\310pbr\301"
    SecureRandom.random_bytes(10) #=> "\323U\030TO\234\357\020\a\337"

Generate alphanumeric strings:

    SecureRandom.alphanumeric(10) #=> "S8baxMJnPl"
    SecureRandom.alphanumeric(10) #=> "aOxAg8BAJe"

Generate UUIDs:

    SecureRandom.uuid #=> "2d931510-d99f-494a-8c67-87feb05e1594"
    SecureRandom.uuid #=> "bad85eb9-0713-4da7-8d36-07a8e4b00eab"

[SecureRandom Reference](https://ruby-doc.org/stdlib-2.7.0/libdoc/securerandom/rdoc/SecureRandom.html)
