When parsing android terminology the following notes may be helpful.

https://source.android.com/security/overview/kernel-security
> Android 6.0 and later supports verified boot and device-mapper-verity. Verified boot guarantees the integrity of the device software starting from a hardware root of trust up to the system partition. During boot, each stage cryptographically verifies the integrity and authenticity of the next stage before executing it.
> Android 7.0 and later supports strictly enforced verified boot, which means compromised devices cannot boot.

https://source.android.com/security/verifiedboot
> In addition to ensuring that devices are running a safe version of Android, Verified Boot checks for the correct version of Android with rollback protection. Rollback protection helps to prevent a possible exploit from becoming persistent by ensuring devices only update to newer versions of Android.
> Android 4.4 added support for Verified Boot and the dm-verity kernel feature. This combination of verifying features served as Verified Boot 1. 
> Where previous versions of Android warned users about device corruption, but still allowed them to boot their devices, Android 7.0 started strictly enforcing Verified Boot to prevent compromised devices from booting. Android 7.0 also added support for forward error correction to improve reliability against non-malicious data corruption.

> When booting in I/O Error (eio) mode, the device shows an error screen informing the user that corruption has been detected and the device may not function correctly. The screen shows until the user dismisses it. In eio mode the dm-verity driver will not restart the device if a verification error is encountered, instead an EIO error is returned and the application needs to deal with the error.

> Android 8.0 and higher includes Android Verified Boot (AVB)


https://source.android.com/compatibility/cdd

> Welcome to the Android Compatibility Definition Document (CDD). This document enumerates the requirements that must be met in order for devices to be compatible with the latest version of Android. 

https://source.android.com/compatibility/6.0/android-6.0-cdd.pdf
see: "9.11. Keys and Credentials"


## Qualcomm

- HLOS = high level operating systems  
  "Qualcomm® Trusted Execution Environment is a controlled and separated environment outside the high-level operating system (HLOS) that is designed to allow trusted execution of code and to protect against viruses, Trojans, and root kits." https://www.qualcomm.com/products/features/mobile-security-solutions
