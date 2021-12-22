
# SSL Certificates part 2
[Part1](https://github.com/PrimozRatej/SSL-Certificates) 
### So we decided to do pinning of rootCA certificates.
1. To install false Ce as RootCA to a device, that device needs to be rooted.
2.  Android emulators already are rooted  [link](https://stackoverflow.com/questions/5095234/how-to-get-root-access-on-android-emulator#:~:text=Please%20note%20that,Chris%20Stratton).
3.  Look like they are  [2 profiles of emulators](https://stackoverflow.com/questions/53662102/adb-root-command-returns-adbd-cannot-run-as-root-in-production-builds-even-o/53662443#:~:text=I%20checked%20some,for%20more%20information.).
4. Ok look like they are not all rooted, to root emulator I follow  [this](https://docs.mitmproxy.org/stable/howto-install-system-trusted-ca-android/#:~:text=Instructions%20for%20API%20LEVEL%20%3E%2028).
5. And now I get [this](https://drive.google.com/file/d/1t3blJA-X9P-Mi2l7LHP0KhKKldxPx9dQ/view?usp=sharing) when running the app.
####   Findings:
- The user can load certificates in rootCA only if he has root access to the device.
- Snooping only works when the user has a rootCA certificate loaded, which means if the device is rooted.  
- We are checking to see if the device has been rooted since 29.11.2021. 
- Version: 34c1bc606246ca477a2ce3f71366499d7832d88e
- However, an attacker can still impose a false cert on a device from Nougat (7.0) or older [source](https://stackoverflow.com/questions/4461360/how-to-install-trusted-ca-certificate-on-android-device).
- Ok so let's check that...
- Nougat 7.0.0 is API level 24 our application is using minSdkVersion 21. so the application should be working [but would it?](https://drive.google.com/file/d/1WJ_khoutmjn6k7Sq5x8FFsDqizqqf5M3/view?usp=sharing).
#### What now?
- So we disabled the possibility for rooted devices to use the application.
- Application was pen tested in October (don't know the exact date), but I assume that the version where we implemented the feature for rooted devices was not included in the tested version.
- If the attacker wants to sniff on traffic it would need a rooted device with a self-sign certificate in the rootCA list, that is Nougat 7.0.0 or older, and look like the application is not working on that old API level (even if the minSdkVersion is 21 Lolipop).
- I could run the app stating on Oreo.
- **So if we disabled rooted devices to use application and application wouldn't run on API level 26 or older. In my eyes, we fixed the Problem: Trying to sniff on app traffic with self-sign certificates.** 

#### But like Franci says: "There is always a hole". 
##### So where is it?
- If the attacker get trusted cert (from any rootCA), then he could intercept traffic, if we are not pinning the rootCA
- If we are pinning the rootCA, then the attacker would need a trusted cert from that specific rootCA.
- If we are pinning leaf cert, we are in both cases protected, but then there is this [question](https://security.stackexchange.com/questions/224246/certficate-pinning-should-i-pin-the-leaf-or-intermediate#:~:text=You%20can%20pin,of%20the%20differences:) (paragraph 3).
- If we are pinning the whole chain is not really a big difference from when we are pinning the leaf.

**But now we are making our problem to not trust SSL**
