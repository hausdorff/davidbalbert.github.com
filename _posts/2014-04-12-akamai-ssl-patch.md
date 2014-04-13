---
layout: post
title: "How does Akamai's \"secure heap\" patch to OpenSSL work?"
permalink: akamai-ssl-patch.html
comments: true
technique: ■
---


For more than a decade, Akamai has guarded their users' private RSA keys using a security-conscious variant of the `malloc` family. In effect, this allows their systems to maintain a second, more secure heap, which makes it significantly harder to execute a broad class of security vulnerabilities.

Yesterday, [Rich Salz](http://en.wikipedia.org/wiki/Rich_Salz) disseminated a patch to `openssl-users` that adds a variation of this `malloc` family to OpenSSL. An archived version of Salz's email is [here](http://www.mail-archive.com/openssl-users@openssl.org/msg73503.html). The effect is that in the forseeable future, it should be possible for OpenSSL to store RSA private keys on a the so-called "secure" heap.

Now, **I know literally nothing about security or systems programming**, but I found this fascinating, and couldn't help but crack it open to see how it worked. In the rest of this post we'll explore the implementation in detail.


## What you'll need

I've gone ahead and forked OpenSSL v1.0.1g, integrated Salz's patch, and [put it on GitHub](https://github.com/hausdorff/openssl-with-secure-malloc) so that it takes a minimal amount of effort to tinker with. To build the patched OpenSSL, simply download the repot and run `./configure` and `make`.


## Basic architecture

Salz describes the patch as adding a "secure arena":

* The arena is an `mmap`'d slice of memory with guard pages allocated before and after. These guard pages are marked `PROT_NONE`, which means that accessing them causes a segmentation fault. This results means that **a wandering pointer will segfault when it accesses memory in this page**, making it easier to protect things in this "secure" heap.
* The arena is resident to memory, and **won't appear on disk.**
* Specifically handles allocation of RSA private keys, and **nothing else.**


## The patch

A nice highlighted GitHub diff of Salz's original patch is in my repository [here](https://github.com/hausdorff/openssl-with-secure-malloc/commit/033f156040d1ff175591c924c2a0c1a0d75ad356). Note that the part that protects the guard pages with `PROT_NONE` actually appears [later in my commit history](https://github.com/hausdorff/openssl-with-secure-malloc/commit/ba8c04147b007b55d9ecab50eb3e2061326fa4fe).

There are two interesting parts of this patch:

* the new `malloc` family that builds the secure heap, and
* the code that causes OpenSSL to store all RSA private keys it receives in this secure heap.

The **tl;dr** of how this is accomplished is:

* OpenSSL normally uses macros in `crypto/crypto.h` to wrap the calls to the `malloc` family. For example `OPENSSL_malloc` wraps the call to `malloc`. Salz begins by refactoring these macros to point at `secure_malloc` and friends instead of the normal `malloc`.
* `secure_malloc` will allocate memory from the secure heap if and only if the secure heap is "turned on". If it's not, it defaults to normal `malloc`. (We'll just ignore `realloc` and `free` because they're less interesting.)
* Salz's patch causes OpenSSL to "turn on" the secure heap immediately after encountering an RSA private key. It turns off the secure heap when it encounters something other than an RSA private key. By switching on and off in this way, the call to `secure_malloc` properly routes RSA private keys to the secure heap, and everything else to the "normal" heap.
* The first time we turn on the secure heap, it is initialized.

If you're following along at home, the interesting parts of this are contained mainly in three files, which you can see in the above diff:

* `crypto/secure_malloc.c`, which contains public api of this new `malloc` family,
* `crypto/buddy_allocator.c`, the code that does most of the allocation work for the new `malloc` family, and
* `crypto/asn1/tasn_dec.c`, the code that handles the receipt and allocation of all RSA private keys.


## Receiving and allocating the RSA private keys on the secure heap

The gist of this section is that OpenSSL uses [ASN.1](http://en.wikipedia.org/wiki/Abstract_Syntax_Notation_One) to encode structured data of various types, including RSA private keys. Since all private keys must come through the function `ASN1_item_ex_d2i` (in the `crypto/asn1/tasn_dec.c` file), we **need only to augment the function to allocate all ASN.1 items that encode RSA private keys on the secure heap instead of the normal heap.** Anything else, in contrast, will go on the normal heap.

If you're not interested in the plumbing of this, you can skip this section. Otherwise a more detailed description follows.

ASN.1 items are sent to the `ASN1_item_ex_d2i` function, which is located on [line 154](https://github.com/hausdorff/openssl-with-secure-malloc/blob/master/crypto/asn1/tasn_dec.c#L154) of my patched version of `crypto/asn1/tasn_dec.c`:

```c
 /* Decode an item, taking care of IMPLICIT tagging, if any.
   * If 'opt' set and tag mismatch return -1 to handle OPTIONAL
   */
  
  int ASN1_item_ex_d2i(ASN1_VALUE **pval, const unsigned char **in, long len,
  			const ASN1_ITEM *it,
  			int tag, int aclass, char opt, ASN1_TLC *ctx);
```

This means that any time we receive an RSA private key, it must come through this function, encoded as an ASN.1 object. Now our task is to simply find all the ASN.1 items that encode RSA private keys, and allocated them on the secure heap instead of the normal heap.

To begin, on [line 173](https://github.com/hausdorff/openssl-with-secure-malloc/blob/master/crypto/asn1/tasn_dec.c#L173) of the `ASN1_item_ex_d2i` function, Salz adds the following local variables, which we will use to track whether the current ASN.1 item contains an RSA private key (rather than, say, and RSA *public* key).

```c
  	int ret = 0;
  	ASN1_VALUE **pchptr, *ptmpval;
 +
 +	int ak_is_rsa_key      = 0; /* Are we parsing an RSA key? */
 +	int ak_is_secure_field = 0; /* should this field be allocated from the secure arena? */
 +	int ak_is_arena_active = 0; /* was the secure arena already activated? */
 +
  	if (!pval)
  		return 0;
  	if (aux && aux->asn1_cb)

```

Then, beginning on [line 417](https://github.com/hausdorff/openssl-with-secure-malloc/blob/master/crypto/asn1/tasn_dec.c#L417) (this is still in the function `ASN1_item_ex_d2i`) Salz adds code to check if the ASN.1 item has `sname` starting with the characters `'R'`, `'S'`, and `'A'`. If this is true, this item encodes either an RSA private key, or an RSA public key. So we set `ak_is_rsa_key = 1`:

```c

 		if (asn1_cb && !asn1_cb(ASN1_OP_D2I_PRE, pval, it, NULL))
  				goto auxerr;
  
 +		/* Watch out for this when OpenSSL is upgraded! */
 +		/* We have to be sure that it->sname will still be "RSA" */
 +		if (it->sname[0] == 'R' && it->sname[1] == 'S' && it->sname[2] == 'A' && it->sname[3] == 0)
 +			ak_is_rsa_key = 1;
 +
  		/* Get each field entry */
  		for (i = 0, tt = it->templates; i < it->tcount; i++, tt++)

```

Finally, starting on [line 469](https://github.com/hausdorff/openssl-with-secure-malloc/blob/master/crypto/asn1/tasn_dec.c#L469), Salz adds code to check whether the ASN.1 template field name starts with any of the following characters: `'d'`, `'p'`, or `'q'`. If so, **this item is our private key**, and it must be allocated on the secure heap.

* So we set `ak_is_secure_field = 1`, indicating this field needs to be allocatedon the secure heap, and
* call `start_secure_allocation`, which will initialize the secure heap if it hasn't been initialized already.

The corresponding code:

```c
 			/* attempt to read in field, allowing each to be
  			 * OPTIONAL */
  
 + 
 +			/* Watch out for this when OpenSSL is upgraded! */
 +			/* We have to be sure that seqtt->field_name will still be */
 +			/* "d", "p", and "q" */
 +			ak_is_secure_field = 0;
 +			ak_is_arena_active = 0;
 +			if (ak_is_rsa_key)
 +			{
 +				/* ak_is_rsa_key is set for public keys too */
 +			        /* however those don't have these variables */
 +				const char *f = seqtt->field_name;
 +				if ((f[0] == 'd' || f[0] == 'p' || f[0] == 'q') && f[1] == 0)
 +				{
 +					ak_is_secure_field = 1;
 +					ak_is_arena_active = start_secure_allocation();
 +				}
 +			}
 +
```

So what happens next? Don't we need to call `secure_malloc` and allocate this RSA private key on the heap?

No! In fact, in `crypto/crypto.h`, we see that Salz changes OpenSSL's core `malloc`-wrapping macro, `OPENSSL_malloc`, to point to `secure_malloc` instead of `CRYPTO_malloc`:

```c
 [...]
 -#define OPENSSL_malloc(num)	CRYPTO_malloc((int)num,__FILE__,__LINE__)
 [...]
 +#define OPENSSL_malloc(s)       secure_malloc(s)
 [...]
```

(NOTE: of course Salz this also redefines all the other family members like `OPENSSL_free` and `OPENSSL_realloc`, not just `malloc`. We've just chosen to omit them here.)

This means that, later in the function `ASN1_item_ex_d2i`, when it is time to save this ASN.1-encoded item, we will call `asn1_enc_save` (which, by the way, is in `crypto/asn1/tasn_utl.c`):

```c
 		/* Save encoding */
  		if (!asn1_enc_save(pval, *in, p - *in, it))
  			goto auxerr;
```

Internally, this will call `OPENSSL_malloc`, but instead of calling the normal `malloc`, we will now call `secure_malloc`, since `OPENSSL_malloc` now points at `secure_malloc`:

```c
int asn1_enc_save(ASN1_VALUE **pval, const unsigned char *in, int inlen,
							 const ASN1_ITEM *it)
	{
[...]
	enc->enc = OPENSSL_malloc(inlen);
	if (!enc->enc)
		return 0;
[...]
	return 1;
	}

```

As we will see, internally our `secure_malloc` will allow us to allocate to the secure heap if and only if the secure heap is initialized; if not, it defaults to normal `malloc`.

This allows us to make the same call and simply change how we allocate based on whether the secure heap is enabled.


## The secure malloc

It turns out that the magic of `secure_malloc` isn't actually in the `malloc` function itself. Like most `malloc`s, `secure_malloc` basically traverses free lists, and peels off some memory to service the request, or returns `NULL` if allocation failed.

The initialization code, on the other hand, is interesting.

We begin with a call to `secure_malloc_init` (in `crypto/buddy_allocator.c`). It takes as arguments `size`, the size in bytes we're to give the secure heap, `mem_min_unit`, which I think is the minimum number of bytes to give an object allocatedin the secure heap, and `overrun_bytes`, which I frankly didn't bother to understand.

```c
/* Module initialization, returns >0 upon success */
int secure_malloc_init(size_t size, int mem_min_unit, int overrun_bytes)
{
[...]
}
```

The interesting part of this function is the central `if`-`else` chain in the middle. We will unpack it in a second.

```c
  if (arena)
  {
    assert(0);
  }
  else if ((arena = (char *) cmm_init(arena_size, mem_min_unit, overrun_bytes)) == NULL)
  {
  }
  else if (mlock(arena, arena_size))
  {
  }
  else if (pthread_key_create(&secure_allocation_key, 0) != 0)
  {
  }
  else
  {
    secure_allocation_support = 1;
    ret = 1;
  }

```

Put succinctly:

* The `if (arena)` block checks to see if the secure heap (or "`arena`") is already built. If it is, we'll return the local variable `ret` which at this point is `0`, indicating that we failed to initialize the secure heap.
* The next block, `else if ((arena = (char *)[...]`, will build the secure heap. We'll see how this works in a second.
* The next block, `else if (mlock(arena, arena_size))`, will lock the secure heap into memory, so that it never goes to disk.
* The next block, `else if (pthread_key_create([...]` will create a thread-specific data "key" we will use to check things like whether secure allocation is enabled.
* The final block, `else`, will set `secure_allocation_block = 1` and set `ret = 1`. When this function returns, it will return `ret`, which if we make it this far, will be larger than 0, which indicates success.

Important to note is that if any of these fails, the `else`-`if` block terminates early and `ret` never gets set to 1, which means we will return reporting a state of error.

So what does the call to `cmm_init` do? This is the part where the secure heap is actually built in memory.

`cmm_init` is in `crypto/buddy_allocator.c`. The declaration looks like the following, taking the same parameters as `secure_malloc_init`.

```c
void *
cmm_init(int size, int mem_min_unit, int overrun_bytes)
{
[...]
}
```

Most of the method is spent allocing enough space for things like free lists. The interesting part comes after this:

```c
    cmm_arena = mmap(NULL, pgsize + mem_arena_size + pgsize, PROT_READ|PROT_WRITE,
		     MAP_ANON|MAP_PRIVATE, 0, 0);

    assert(MAP_FAILED  != cmm_arena);
    mprotect(cmm_arena, pgsize, PROT_NONE);
    mprotect(cmm_arena + aligned, pgsize, PROT_NONE);

[...]

    return cmm_arena;

```

In the first line, we're using `mmap` to allocate space requested for the secure heap (denoted as `mem_arena_size`), plus space for one page on either side of the secure heap (denoted by the variable `pgsize`). Because we pass in `NULL` as the first parameter &mdash; normally it's an address &mdash; the kernel will just put this memory wherever it wants. This space is readable and writable. The `MAP_ANON` and `MAP_PRIVATE` flags mean that the memory mapping does not correspond to a file, and when it is written, it should never be attached to a file.

The next two lines are calls to `mprotect`. They are essentially ensuring that the guard pages on either side of the secure heap are marked `PROT_NONE`, which means you will segfault if you try to access them. Here the variable `aligned` denotes the total size of the secure heap between the guard pages (which if you look at the function itself, is rounded up to the nearest page).

We return `cmm_arena` in the last line.


## Conclusions

The end result of the initialization phase is more or less what Salz promised. The secure heap is pinned to memory, and the guard pages cause segfaults if you accidentally access them.

If you're interested to see how the rest of the system works, the `malloc`, `realloc`, `free`, *etc*. are all worth a read, but they're not especially crazy as `malloc` implementations go.

One interesting thing to note: I couldn't find anywhere that `secure_malloc_init` was actually called in the code. This means that it's never actually being initialized, and therefore never being used. Or I'm missing something.