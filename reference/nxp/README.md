NXP secure boot implementations references and some notes.

**nxp/hab**

- Secure Boot on i.MX50, i.MX53, i.MX 6 and i.MX7 Series using HABv4 
  [local](nxp/hab/AN4581.pdf) 
  [source](https://www.nxp.com/docs/en/application-note/AN4581.pdf)  
  Figure 1 page 4 and Figure 5 page 33 describes zImage format  
- https://boundarydevices.com/high-assurance-boot-hab-dummies/
- The i.MX Code Signing Tool is used to generate the HABv4 signatures for images using the PKI tree data and SRK table. 
- On the target, HAB evaluates the SRK table included in the signature by hashing it and comparing the result to the SRK fuse values. 
- If the SRK verification is successful, this establishes the root of trust, 
- and the remainder of the signature can be processed to authenticate the image.

**nxp/hap Quarkslab vulnerabilities**  
  https://blog.quarkslab.com/vulnerabilities-in-high-assurance-boot-of-nxp-imx-microprocessors.html
> Stack-based buffer overflow in the X.509 certificate parser (CVE-2017-7932)
> can be exploited by loading a crafted certificate
> ... Hence, the entire certificate is parsed before having its signature verified.
> Buffer overflow in SDP recovery mode (CVE-2017-7936)  
  
* Effected platforms:  
  https://nvd.nist.gov/vuln/detail/CVE-2017-7932  
  NXP i.MX 28 i.MX 50, i.MX 53, i.MX 7Solo i.MX 7Dual Vybrid VF3xx, Vybrid VF5xx, Vybrid VF6xx, i.MX 6ULL, i.MX 6UltraLite, i.MX 6SoloLite, i.MX 6Solo, i.MX 6DualLite, i.MX 6SoloX, i.MX 6Dual, i.MX 6Quad, i.MX 6DualPlus, and i.MX 6QuadPlus.  
  https://nvd.nist.gov/vuln/detail/CVE-2017-7936  
  NXP i.MX 50, i.MX 53, i.MX 6ULL, i.MX 6UltraLite, i.MX 6SoloLite, i.MX 6Solo, i.MX 6DualLite, i.MX 6SoloX, i.MX 6Dual, i.MX 6Quad, i.MX 6DualPlus, i.MX 6QuadPlus, Vybrid VF3xx, Vybrid VF5xx, and Vybrid VF6xx. 
* Inverse path PoC on USB Armory:  
  https://github.com/inversepath/usbarmory/blob/master/software/secure_boot/usbarmory_csftool#L227  
* NXP possible mitigations:  
  https://community.nxp.com/docs/DOC-334996  
  "i.MX & Vybrid Security Vulnerability Errata - ERR010872, ERR010873"  
  > An Engineering Bulletin (EB00854) on possible mitigation strategies is available to impacted users and can be requested through your NXP field support team or distributor. 

  Could not obtain EB00854 document. But attached to the NXP bulletin is additional information:  
  https://community.nxp.com/servlet/JiveServlet/download/334996-3-406868/ERR010873_Secure_Boot_Vulnerability_Erratum_Preliminary_Rev0.pdf  
  > There is no software workaround available to prevent this vulnerability for the affected devices because the vulnerability is in the Boot ROM which cannot be updated in the field.  

  https://community.nxp.com/servlet/JiveServlet/download/334996-3-406867/ERR010872_Secure_Boot_Vulnerability_Erratum_Preliminary_Rev0.pdf  
  > For the i.MX 6UltraLite and i.MX 6ULL devices, a customer programmable eFUSE is available to disable the SDP port,thereby completely preventing this vulnerability
- It is possible to mitigate one or both of the 
  secure boot vulnerabilities using a modified boot image. This image can be
  obtained from NXP and all future releases will mitigate the issue.

**nxp/prince**

- Secure Boot on LPC55xx/LPC55Sxx
  LPC55xx/LPC55Sxx is an Arm Cortex ® M33-based microcontroller
  [local](nxp/prince/AN12283.pdf)
  [source](https://www.nxp.com/docs/en/application-note/AN12283.pdf)
  > "Secure boot with signed image, boot from encrypted PRINCE flash region, and secure boot with device identifier composition engine (DICE)."

  ![unsigned image](nxp/prince/boot_image_unsigned-plain.png)
  ![unsigned image crc](nxp/prince/boot_image_unsigned-plain-crc.png)
  ![unsigned image signed](nxp/prince/boot_image_signed.png)
  >Image validation is a two-step process. The first step is the validation of the X.509 certificate inserted in the image. This contains the image public key used in the second step to validate the entire image (including the certificate) to allow customers to add additional PKI structure.
  ![flash region prince encrypted](nxp/prince/flash_region_prince_encrypted.png)

  > LPC55Sxx supports on-the-fly encryption/decryption to/from internal flash through PRINCE.
  > Each PRINCE region has a secret-key supplied from on-chip SRAM PUF via secret-bus interface (not SW accessible)

  ![rom region prince protected](nxp/prince/rom_region_prince_protected.png)
  ```
    Table 1. SECURE_BOOT_CFG address 0x9E41C
    Bit     Symbol
    29:0    Reserved
    31:30   SEC_BOOT_EN
            Value   Description
            0b00    Plain image
            0b01    Signed image
            0b10    Signed image
            0b11    Signed image```
            
- supports DeviceIdentifier Composition Engine (DICE) from Trusted Computing Group
- supports PRINCE on-the-fly flash encryption/decryption engine

- PRINCE Challenge  
  https://en.wikipedia.org/wiki/Prince_(cipher)
  > To encourage cryptoanalysis of the Prince cipher, the organizations behind it created the "Prince challenge".
  
  Various (read many) attacks are described. Seems if you find DES unacceptable
  then PRINCE should be as well.