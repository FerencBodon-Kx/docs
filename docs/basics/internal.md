---
title: Internal functions | Language | kdb+ and q documentation
description: The operator ! with a negative integer as left-argument calls an internal function.
---
# :fontawesome-solid-triangle-exclamation: Internal functions




The operator `!` with a negative integer as left argument calls an internal function.

<div markdown="1" class="typewriter">
[0N!x](#0nx-show)        show                          Replaced:
[-4!x](#-4x-tokens)        tokens                        -1!   [hsym](../ref/hsym.md)
[-8!x](#-8x-to-bytes)        to bytes                      -2!   [attr](../ref/attr.md)
[-9!x](#-9x-from-bytes)        from bytes                    -3!   [.Q.s1](../ref/dotq.md#s1-string-representation)
[-10!x](#-10x-type-enum)       type enum                     -5!   [parse](../ref/parse.md)
[-11!](#-11-streaming-execute)        streaming execute             -6!   [eval](../ref/eval.md)
[-14!x](#-14x-quote-escape)       quote escape                  -7!   [hcount](../ref/hcount.md)
[-16!x](#-16x-ref-count)       ref count                     -12!  [.Q.host](../ref/dotq.md#host-ip-to-hostname)
[-18!x](#-18x-compress-bytes)       compress bytes                -13!  [.Q.addr](../ref/dotq.md#addr-iphost-as-int)
[-21!x](#-21x-compressionencryption-stats)       compression/encryption stats  -15!  [md5](../ref/md5.md)
[-22!x](#-22x-uncompressed-length)       uncompressed length           -19!  [set](../ref/get.md#set)
[-23!x](#-23x-memory-map)       memory map                    -20!  [.Q.gc](../ref/dotq.md#gc-garbage-collect)
[-25!x](#-25x-async-broadcast)       async broadcast               -24!  [reval](../ref/eval.md#reval)
[-26!x](#-26x-ssl)       SSL                           -29!  [.j.k](../ref/dotj.md#jk-deserialize)
[-27!(x;y)](#-27xy-ieee754-precision-format)   format                        -31!  [.j.jd](../ref/dotj.md#jjd-serialize-infinity)
[-30!x](#-30x-deferred-response)       deferred response             -32!  [.Q.btoa](../ref/dotq.md#btoa-b64-encode)
[-33!x](#-33x-sha-1-hash)       SHA-1 hash                    -34!  [.Q.ts](../ref/dotq.md#ts-time-and-space)
[-36!](#-36-load-master-key)        load master key               -35!  [.Q.gz](../ref/dotq.md#gz-gzip)
[-38!x](#-38x-socket-table)       socket table                  -37!  [.Q.prf0](../ref/dotq.md#prf0-code-profiler)
[-120!x](#-120x-memory-domain)      memory domain
</div>

!!! warning "Internal functions are for use by language implementors."

    They are [exposed infrastructure](exposed-infrastructure.md)
    and may be redefined in subsequent releases.

    They also allow new language features to be tried on a provisional basis.

    Where they are replaced by keywords or utilities, **use the replacements**.

[![Neal Stephenson thinks it’s cute to name his labels 'dengo'](../img/goto.png "Neal Stephenson thinks it’s cute to name his labels 'dengo'")](https://xkcd.com/292/)
_xkcd.com_


## `0N!x` (show)

The identity function.
Returns `x` after writing it to the console.

An essential tool for debugging.


## `-4!x` (tokens)

Returns the list of q tokens found in string `x`. (Note the q parsing of names with embedded underscores.)

```q
q)-4!"select this from that"
"select"
," "
"this"
," "
"from"
," "
"that"

q)-5!"select this from that" / compare with −5!
?
`that
()
0b
(,`this)!,`this

q)-4!"a variable named aa_bb"
,"a"
," "
"variable"
," "
"named"
," "
"aa_bb"
q)
```

!!! warning 

    Should not be used with input data over 2GB in length (0Wi). Returns domain error with this condition since 4.1 2022.04.15.

## `-8!x` (to bytes)

Returns the IPC byte representation of `x`.

```q
q)-8!1 2 3
0x010000001a000000060003000000010000000200000003000000
```

:fontawesome-solid-hand-point-right:
[`-9!x`](#-9x-from-bytes) (from bytes)


## `-9!x` (from bytes)

Creates data from IPC byte representation `x`.

```q
q)-9!-8!1 2 3
1 2 3
```

:fontawesome-solid-hand-point-right:
[`-8!x`](#-8x-to-bytes) (to bytes), [-18!x](#-18x-compress-bytes) (compress bytes)

## `-10!x` (type enum)

Resolve a [type](datatypes.md) number to an [enum](enumerations.md) vector and check if it is available.

```q
q)-10!20h
1b
q)ee:`a`b`c
q)vv:`ee$`a`a`b
q)type vv
20h
q)-10!20h
0b
```


## `-11!` (streaming execute)

Replay the q interpreter on messages stored in a log file.

```syntax
-11!x
-11!(-1;x)
-11!(-2;x)
-11!(n;x)
```

Where `n` is a non-negative integer and `x` is a logfile handle

`-11!x` and `-11!(-1;x)`

: replay `x` and return the number of chunks executed; if end of file is corrupted, signal `badtail`.

`-11!(-2;x)`

: returns the number of consecutive valid chunks in `x` and the length of the valid part of the file

`-11!(n;x)`

: replays `n` chunks from top of logfile and returns the number of chunks executed

Each chunk from a log is passed to [`.z.ps`](../ref/dotz.md#zps-set) for execution.
In replaying, if the logfile references an undefined function, the function name is signalled as an error.

:fontawesome-solid-graduation-cap:
[Log files](../kb/logging.md)


## `-14!x` (quote escape)

Handles `"` escaping in strings: used to prepare data for CSV export.


## `-16!x` (ref count)

Returns the reference count for a variable.

```q
q)-16!a
1
q)a:b:c:d:e:1 2 3
q)-16!a
5
```


## `-18!x` (compress bytes)

Returns the IPC byte representation of `x` according to [`-8!x`](#-8x-to-bytes), applying compression based on [IPC compression rules](ipc.md#compression):

* Uncompressed serialized data has a length greater than 2000 bytes
* Size of compressed data is less than &frac12; the size of uncompressed data

```q
q)count -8!til 1000     / uncompressed
8014
q)count -18!til 1000    / compressed
3276
```

[-9!x](#-9x-from-bytes) can be used to uncompress and deserialise.

```q
q)a:til 1000           / original data to convert 
q)x:-18!a              / serialize and compression to bytes using IPC serialisation
q)a~-9!x               / test if deserialised version is same as original
1b
```

:fontawesome-solid-hand-point-right:
[`-22!x`](#-22x-uncompressed-length) (uncompressed length), [`-9!x`](#-9x-from-bytes) (from bytes), [.Q.gz](../ref/dotq.md#gz-gzip) (gzip)

<!-- 
## `-19!` (compress file)

```txt
-19!(src;tgt;lbs;alg;lvl)
```

Where

-   `src` is the source file (filesymbol)
-   `tgt` is the target file or folder (filesymbol)
-   `lbs` is logical block size (long)
-   `alg` is compression algorithm (long)
-   `lvl` is compression level (long)

reads `src`, writes it compressed to `tgt`, and returns `tgt`.

```q
q)`:test set asc 10000000?100; / create a test data file
`:test
q)/ compress input file test, to output file ztest
q)/ using a block size of 128kB (2 xexp 17), gzip level 6
q)-19!(`:test;`:ztest;17;2;6)
99.87667
q)/ check the compressed data is the same as the uncompressed data
q)get[`:test]~get`:ztest
1b
```

:fontawesome-solid-database:
[File compression](../kb/file-compression.md)
<br>
:fontawesome-solid-book:
[`.z.zd` zip defaults](../ref/dotz.md#zzd-compressionencryption-defaults)
 -->

[](){#-21x-compression-stats}
## `-21!x` (compression/encryption stats)

Where `x` is a file symbol, returns a dictionary of compression/encryption statistics for it. Encryption available since 4.0 2019.12.12.
The dictionary is empty if the file is not compressed/encrypted.

```q
q)-21!`:ztest       / compressed
compressedLength  | 137349
uncompressedLength| 80000016
algorithm         | 2i
logicalBlockSize  | 17i
zipLevel          | 6i
q)-21!`:test        / not compressed
q)count -21!`:test
0
q)-21!`:ztest       / encrypted
compressedLength  | 40088
uncompressedLength| 40008
algorithm         | 16i
logicalBlockSize  | 17i
zipLevel          | 6i
```

:fontawesome-solid-book:
[`set`](../ref/get.md#set)
<br>
:fontawesome-solid-database:
[File compression](../kb/file-compression.md)
<br>
:fontawesome-solid-database:
[Data at rest encryption (DARE)](../kb/dare.md)


## `-22!x` (uncompressed length)

An optimized shortcut to obtain the length of uncompressed serialized `x`, i.e. `count -8!x`

```q
q)v:til 100000
q)\t do[5000;-22!v]
1
q)\t do[5000;count -8!v]
226
q)(-22!v)=count -8!v
1b
```

:fontawesome-solid-hand-point-right:
[`-18!x`](#-18x-compress-bytes) (compress bytes)


## `-23!x` (memory map)

Since V3.1t 2013.03.04

Attempts to force the object `x` to be resident in memory by hinting to the OS and/or faulting the underlying memory pages.
Useful for triggering sequential access to the storage backing `x`.


## `-25!x` (async broadcast)

Since V3.4

Broadcast data as an async msg to specified handles. The advantage of using `-25!(handles;msg)` over `neg[handles]@\:msg` is that `-25!msg` will serialize `msg` just once – thereby reducing CPU and memory load.

Use as

```q
q)-25!(handles; msg)
```

Handles should be a vector of positive int or longs.

`msg` will be serialized just once, to the lowest capability of the list of handles. I.e. if handles are connected to a mix of versions of kdb+, it will serialize limited to the types supported by the lowest version. If there is an error, no messages will have been sent, and it will return the handle whose cap caused the error.

Just as with `neg[handles]@\:msg`, `-25!x` queues the msg as async on those handles – they don't get sent until the next spin of the main loop, or are flushed with `neg[handles]@\:(::)`.

!!! tip "`-25!(handles; ::)` can also flush the handles"

Possible error scenarios:

-   from trying to serialize data for a handle whose remote end does not support a type, or size of the data.

    ```q
    / connect to 2.8 and 3.4
    q)h:hopen each 5000 5001
    q)h
    5 6i 
    q)(-5) 0Ng / 2.8 does not support guid
    'type
    q)(-6) 0Ng / 3.4 does support guid 
    q)-25!(h;0Ng)
    'type error serializing for handle 5
    ```

-   an int is passed which is not a handle

    ```q
    q)-25!(7 8;0Ng)
    '7 is not an ipc handle
    ```



## `-26!x` (SSL)

View TLS settings on a handle or current process `-26!handle` or `-26!()`.
Since V3.4 2016.05.12.

```q
q)(-26!)[]
SSLEAY_VERSION   | OpenSSL 1.0.2g  1 Mar 2016
SSL_CERT_FILE    | /Users/kdb/certs/server-crt.pem
SSL_CA_CERT_FILE | /Users/kdb/certs/ca.pem
SSL_CA_CERT_PATH | /Users/kdb/certs/
SSL_KEY_FILE     | /Users/kdb/certs/server-key.pem
SSL_CIPHER_LIST  | ALL
SSL_VERIFY_CLIENT| NO
SSL_VERIFY_SERVER| YES
```

In the result, all keys except `SSLEAY_VERSION` are initialized from their corresponding environment variables.

:fontawesome-solid-hand-point-right:
[`.z.e` TLS connection status](../ref/dotz.md#ze-tls-connection-status)
<br>
:fontawesome-solid-graduation-cap:
[SSL](../kb/ssl.md)


## `-27!(x;y)` (IEEE754 precision format)

Where

-   `x` is an int atom
-   `y` is a float

returns `y` as a string or strings formatted as a float to `x` decimal places.
(Since V3.6 2018.09.26.)
It is atomic and doesn’t take [`\P`](syscmds.md#p-precision) into account. For example:

```q
q)-27!(3i;0 1+123456789.4567)
"123456789.457"
"123456790.457"
```

This is a more precise, built-in version of [`.Q.f`](../ref/dotq.md#f-precision-format) but uses IEEE754 rounding:

```q
q).045
0.044999999999999998
q)-27!(2i;.045)
"0.04"
q).Q.f[2;.045]
"0.05"
```

You might want to apply a rounding before applying `-27!`.


## `-30!x` (deferred response)

Defer response to a sync message. Since V3.6 2018.05.18.

`-30!(::)` 

: allows the currently-executing callback to complete without responding to the client, for example [`.z.pg`](../ref/dotz.md#zpg-get). The handle to use for the subsequent deferred reply can be obtained via [`.z.w`](../ref/dotz.md#zw-handle). The deferred reply should be provided later via one of the following methods:

`-30!(handle;1b;errorMsg)` 

: responds to the deferred sync call with an error message populated with the string/symbol provided in `errorMsg`

`-30!(handle;0b;msg)` 

: responds to the deferred sync call with the contents of `msg`

A `'domain` error will returned if the handle is not  a member of [`.z.W`](../ref/dotz.md#zw-handles).

Using a handle that is not expecting a response message will return an error, for example:
```q
q)key .z.W / list of socket handles being monitored by kdb+ main thread
, 8i
q)-30!(8i;0b;`hello`world) / try to send a response of (0b;`hello`world)
'Handle 8 was not expecting a response msg
  [0]  -30!(8i;0b;`hello`world)
          ^
```

:fontawesome-solid-graduation-cap:
[Deferred response](../kb/deferred-response.md)


## `-33!x` (SHA-1 hash)

```syntax
-33!x
```

where `x` is a string, returns its SHA-1 hash as a list of strings of hex codes.

```q
q)raze string -33!"mypassword"
"91dfd9ddb4198affc5c194cd8ce6d338fde470e2"
```

:fontawesome-solid-book-open:
Command-line options [`-u`](cmdline.md#-u-usr-pwd-local) and [`-U`](cmdline.md#-u-usr-pwd)


## `-36!` Load master key

```syntax
-36!(::)    / since 4.1 2024.03.12 and 4.0 2024.03.02
-36!(x;y)
-36!(x;y;z) / since 4.1 2024.03.12 and 4.0 2024.03.02
```

Where

-   `x` is a master-key file as a [file symbol](glossary.md#file-symbol)
-   `y` is a password as a string
-   `z` is whether to unlock/lock as a bool

`-36!(::)`

Expose whether a key has already been loaded, returning 0b or 1b accordingly.

`-36!(x;y) and -36!(x;y;z)`

loads and validates the master key into memory as the key to use when decrypting or encrypting data on disk.

:fontawesome-solid-graduation-cap:
[Create master key](../kb/dare.md#configuration)

Expect this call to take about 500 milliseconds to execute.
It can be executed from handle 0 only.

Signals errors:
```txt
Encryption lib unavailable      failed to load OpenSSL libs
Invalid password
Main thread only                can be executed from the main thread only
PKCS5_PBKDF2_HMAC               library invocation failed
Restricted                      must be executed under handle 0
Unrecognized key format         master key file format unrecognized
```

`z` indicates unlock/lock. To reload using a new key, unlock using current key and then proceed with the new key. 
If load is attempted while locked, it throws `'DARE key locked`.


## `-38!x` (socket table)
```syntax
-38!x
```

where `x` is a list of socket handles, returns a table with columns

-   `p` (protocol): `q` (IPC) or `w` (WebSocket)
-   `f` (family): `t` (TCP) or `u` (Unix domain socket)
-   `z` (compression enabled flag): since v4.1 2024.05.31
-   `n` (count unsent msgs): since v4.1 2024.05.31
-   `m` (total unsent bytes, like [.z.W](../ref/dotz.md#zw-handles)): since v4.1 2024.05.31

Since v4.0 2020.06.01.

```q
q)h:hopen 5000
q)-38!h
p| "q"
f| "t"
z| 0b
n| 0
m| 0
q){([]h)!-38!h:.z.H}[]
h| p f z n m
-| ---------
8| q u 0 0 0
9| q t 0 0 0
```

:fontawesome-solid-hand-point-right:
[`.z.H` active sockets](../ref/dotz.md#zh-active-sockets), [`.z.W` handles](../ref/dotz.md#zw-handles), [`.z.w` handle](../ref/dotz.md#zw-handle)

## `-120!x` (memory domain)

```syntax
-120!x
```

returns `x`’s memory domain (currently 0 or 1), e.g.

```q
q)-120!'(1 2 3;.m.x:1 2 3)
0 1
```

:fontawesome-solid-book:
[`.m` namespace](../ref/dotm.md)
