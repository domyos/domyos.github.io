Title: Small weiner writeup
Description: A simple Weiner Attack on an RSA encrypted message
Date: 2022-03-14
Category: CTF writeups
Tags: cryptography

# Cryptography
## small weiner
<img width="506" alt="grafik" src="https://user-images.githubusercontent.com/3590155/158152121-365835bd-db30-4d05-b63c-65197af06b7c.png">

To begin we start by search for ways to attack RSA. With some search engine magic we find the [following article](https://www.utc.edu/sites/default/files/2021-04/course-paper-5600-rsa.pdf).
This mentions an attack on low private exponents. By coincidence this attack on small ds was initially described M. Wiener, which perfectly matches the challenge name.

When trying to understand the attack I also found the following papers which are an interesting read:

[Partial Key Exposure Attack On Low-Exponent RSA](https://repository.root-me.org/Cryptographie/EN%20-%20Partial%20Key%20Exposure%20AttackOn%20Low-Exponent%20RSA%20-%20Eric%20W%20Everstine.pdf)

[Cryptanalysis of Short RSA Secret Exponents](https://www.cits.ruhr-uni-bochum.de/imperia/md/content/may/krypto2ss08/shortsecretexponents.pdf)

And there exists also the following video by Jeff Suzuki on YouTube where he describes how Wieners attack works:
[Wieners attack](https://www.youtube.com/watch?v=OpPrrndyYNU)

Since I didn't want to implement the attack myself I googled a little further and found the [goRsaTool](https://github.com/sourcekris/goRsaTool) on GitHub.
This tool can execute Wieners attack.

I then simply created a file `numbers.txt` that contains N and e from the challenge description.

``` bash
# cat numbers.txt
n = 0x26553fbb7e4bd5bd48868a25f24d9cc5975aa8597f82110058e687dfa10dd0114c0d2011fa288dbd9d01c0a70dfa8212d5a218d513bdd8ebed9f75bc299e1461be8a23ed8ade96bc449d409fbbf5a328ee2ad3257e6c55a97641258730f74f4d3938f0df794546791ba2b1518b8d855e83f65f885d67aa000a01687ac605404e7bca681e51e6e195f77eb4785fcda0372e3d0fd90240f736243584677f89da4c6ab54d687897d5afb0801cc151c516b072aaa2d9aa8d39d34c230536cba077beaa88ff8e8940a5ba990cafd0b1326f209873a43a785d0c5477241fb6469b8c27c7d54908467a7525de18b2425901c0de3ed63472831c29818ce6efb0354c61f36b2e61146472e99209d198bc885ced0edb66eab62a968c9b98b49b756c689d69820ca1d97e1232c338084097078265ce79b25c1e37bc777247af3fee2ce7a87a697a120c0428327177cf6e934aa2d18e696474227d361a5c36992788c3b1aa8654b88852e897027d58b21576b25a5ffdcb9fbdc5167eb74f1c9082ae79ca0b89
e = 0xfc2e4d12eb69a42c074d9a0ddc6b84294f1e23d6eaa0ba53e9cb60ec0db203d31bdfb90eaca38189890ad26335ad6107cd234a415bfc73fc1bbd6c5d9da65249eebb57d889f91719cfdbd535ab19d2d317ffdf075870a62c6e05aac16c9b122e1c52d7dbeb2fb683514d0f463b58a4217f2e379e5a62be06e764e043a0eac5ac6af56816af926bcc4cd826ee1cfd4157496dc024042676503cec93de45c3c5e4dd9dcf85406a3cf93a9f784b9eef6e320cd9856aefff48df52127b98da8a0d207f588ce1c58e47419554590b1fa7fa3c38034f93a3a5112b6dd5e78c181abc2d972fbcb058575789c68c03f043bd4bf48d94fa7390c77f9fc033f3f01a5162d31056eb42a07397f3485b25396f93558466fc49ef80adea1e9d6c3d9edf529be5faf014669ae5f8e02433a2474d9c92fcc468d81aa0fd641a5647d55153713783a9e5d66fe70c9c2794325b28f20b751fb49359c4a8487bbfa7efc6270b7fa0ffe277276bba14027596d129fcbdef0a82aba24855bfd2155071b52c11da2d943
```

Then I run `./gorsatool -key numbers.txt -attack wiener -verbose`

![image](https://user-images.githubusercontent.com/3590155/158155106-d85a90cc-91b8-41ac-8273-fb4e2441954f.png)

This seemed to work but the output doesn't match the required d in base10.
So add the line `log.Printf("found the d: %s", c[1])` to output the value of d.

![image](https://user-images.githubusercontent.com/3590155/158157020-c810e966-ff13-4f86-8139-70665aed84a2.png)

This results in the following output:
![image](https://user-images.githubusercontent.com/3590155/158157152-3f8c5db5-80d5-40df-8e7c-d061a861cf09.png)

So the flag is **`dvCTF{79070855007994582698354011721316587208400326157509581241514418985973605934731}`**

