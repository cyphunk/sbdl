# Secure Boot Description Language

-   _v 0.1 - 2019.03.04: Initial publication_ 

## Introduction

The Secure Boot Description Language is used to document the chain-of-trust in
secure boot implementations. Describing a secure boot design in the abstract may
be useful when reviewing the security assurances of an embedded design or can be
used by developers to understand how their design will be perceived to security
analysts. Nearly all secure-boot designs include portions of untrusted memory
and their location should be clearly demarcated to developers to avoid
inadvertently introducing vulnerabilities in the secure boot designs through
software and system configuration flaws.

The initial motivation for writing this language was out of necessity from
conducting embedded security reviews of embedded products. There are many secure
boot reference designs from different vendors and some product vendors may write
their own design from scratch. Each case will use a language to describe their
chain-of-trust, if described at all. These designs are often very similar to
each other and their similarity may only become apparent after generalization. 
This SBDL is intended to help with generalization. Ideally common implementations
can be shared in this repository to make understanding bespoke and new designs
easier. (please contribute)

The language examples given err on the side of being less device specific to
allow greater comparability and temporal accessibility. E.g. ``flash`` is used
to describe memories external to the CPU. This is sufficient for single memory
systems but authors describing multi-memory systems may wish to trade
portability for additional clarity by using ``nand/nor/emmc/usb`` to describe
memory locations.

## Important

This generalization language might obscure important details. For example missing 
are authentication modes and mac functions. Additionally a glaring omission in 
is lack of effective representation for design flaws that occur when decryption 
is assumed to equal authentication. Hopefully readers that find these details 
important could help organize this language to better suite those needs.

## Use 

Here is a simple example showing 1 boot stage, a system kernel, filesystem,
product applications and then an untrusted configuration partition. 
These are secured using cryptographic authentication that depend on keys 
stored internally to the CPU.

**chainoftrust_verified_simple.sbdl**

        boot1 = cpu:rom://boot1
       kernel = boot1:verify(cpu:otp://key, flash://kernel_sign)
     keystore = kernel:verifydecrypt(cpu:otp://key, flash://keystore_encsign)
      filesys = kernel:verify(keystore://key, flash://filesys_sign)
         apps = filesys://applications
    ^^^^^^^^^^^^ trusted ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    vvvvvvvvvvvv untrusted vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
         conf = flash://conf

The ``^^^ trusted ^^^`` and ``vvv untrusted vvv`` delimiters are unnecessary.
The untrusted elements in the SBDL definition should be obvious without these
delimiters but they can be included as a visual warning that indicates where the
chain-of-trust stops.

Elements:

* ``boot1 = cpu:rom://boot1``  
  1st stage bootloader is loaded from ROM stored internally to the CPU. 
  As this is Read Only Memory this stage is trusted.
* ``kernel = boot1:verify(cpu:otp://key, flash://kernel_sign)``  
  OS Kernel stored in flash and verified by 1st stage bootloader software using 
  a key stored in the CPU's One Time Programmable memory.
* ``keystore = kernel:verifydecrypt(cpu:otp://key, flash://keystore_encsign)``  
  A keystore is stored signed and encrypted on flash and decrypted and verified 
  by OS Kernel using a key stored in the CPU's One Time Programmable Memory.
* ``filesys = kernel:verify(keystore://key, flash://filesys_sign)``  
  Filesystem stored in flash and verified by OS kernel using a key
  found in the keystore.
* ``apps = filesys://applications``  
  Applications are loaded from the trusted fileystem.
* ``conf = flash://conf``  
  A configuration partition is stored on flash and loaded without any
  authentication and is therefor untrusted.

Note: the use of ``cpu:otp://key`` is generalized and in actuality different 
OTP keys may be used to refer to different keys the SBDL definition. This 
generalization will still relay important information about trust and key 
storage but if key serialization were important then author may wish to number 
each key.

## Examples

### chainoftrust_encrypted.sbdl

        boot1 = cpu:rom://boot1
        boot2 = boot1:decrypt(cpu:otp://key, flash://boot2_enc)
       kernel = boot2:decrypt(cpu:otp://key, flash://kernel_enc)
     keystore = boot2:decrypt(cpu:otp://key, flash://keystore_enc)
      filesys = boot2:decrypt(keystore://key, flash://filesys_enc)
         apps = filesys://applications
    ^^^^^^^^^^^^ hidden ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    vvvvvvvvvvvv clear vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
         conf = flash://conf

Note: Encryption may provide secrecy but without authentication it should not be assumed to prevent modification.

### chainoftrust_signed.sbdl

       boot1 = cpu:rom://boot1
       boot2 = boot1:verify(cpu:otp://key, flash://boot2_enc)
      kernel = boot2:verify(cpu:otp://key, flash://kernel_enc)
     filesys = boot2:verify(cpu:otp://key, flash://filesys_enc)
        apps = filesys://applications
    ^^^^^^^^^^^^ trusted ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    vvvvvvvvvvvv untrusted vvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvvv
        conf = flash://conf


### chainoftrust_encrypted_signed_keys_on_coprocessor-enclave.sbdl 

       boot1 = cpu:rom://boot1
       cocpu = cpu:coprocessor         #coprocessor is cpu enclave
       boot2 = cocpu:decrypt(cocpu:key_index, flash://boot2_enc)
      kernel = cocpu:decrypt(cocpu:key_index, flash://kernel_enc)
     filesys = boot2:decrypt(cocpu:key_index, flash://filesys_enc)
        apps = filesys://applications
        conf = boot2:verify(cocpu:key_index, flash://conf_signed)

### chainoftrust_dmcrypt_verify.sbdl

    Flash Partitions:
    flash:boot1, flash:boot2, flash:app1, flash:app1, flash:user
    For now all just referenced with flash://

       boot1 = cpu:rom://boot1
       boot2 = boot1:verify(cpu:otp://pubkey, flash://uboot2)
      kernel = boot2:verify(cpu:otp://pubkey, flash://kernel)
     # uses linux dm-verify which auto verifies blocks as loaded
     filesys = kernel:dm-verify(cpu:otp://pubkey, flash://filesys)
    ^^^^^^^^^^^^ trusted ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

### chainoftrust_nxp-hab.sbdl

       boot1 = cpu:rom://boot1
       boot2 = boot1:decrypt(cpu:otp://key, flash://boot2_enc)
      kernel = boot2:decrypt(cpu:otp://key, flash://kernel_enc)
     filesys = flash://filesystem
        apps = filesys://applications


## Contributions

Pull requests welcome.

## Disclaimer

There exist other generalization languages in the cryptographic field but these
are often meant to answer more than the very basic questions required for 
initial secure boot review. Namely the primary questions a secure boot 
generalization should answer are where secrets are stored and where code is 
loaded from. The language should be simple enough to look at one page and
require a minimal of domain specific knowledge. The use of the HTTP ``://`` 
mnemonic is read as:   

  ``thing contains : this thing that stores :// final thing``
   
This notation style could be extended to more literally indicate the access 
protocols used (eg. i2c, spi). Adding access protocol information introduces
additional information density which may reduce comparability and temporal 
portability. The current notation places emphasis on storage locations.