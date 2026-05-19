With April '26 Windows Update, Win11 users received a new RDP client which shows new security warnings when using Remote Apps. 

These warnings cannot be easily dismissed as in the past, and appear each time a Remote App is started. Useful the first time, boring the others.

Since the warnings are related to using *unsigned* `.rdp` files, the most elegant solution is indeed to accept Microsoft take on the problem, sign the `.rdp`s and then use a policy to declare that the signed files are safe and can bypass Microsoft's security prompts.

Luckily, a self signed certificate is enough to sign the `.rdp`s, although it will require the extra step of making it trusted on users' PCs.

Overall, the procedure is the following:

- if you don't have one already, create or buy a new certificate with DigitalSignature usage. 

  You can for example use the 'classical' Powershell:
  ```
  New-SelfSignedCertificate `
    -KeyUsage DigitalSignature `
    -KeyLength 2048 `
    -KeyAlgorithm 'RSA' `
    -TextExtension @('2.5.29.37={text}1.3.6.1.5.5.7.3.2,1.3.6.1.5.5.7.3.1') `
    -Subject 'RDP Signing' `
    -NotAfter (Get-Date).AddYears(10) `
    -CertStoreLocation 'Cert:\CurrentUser\My'
  ```

- (in Powershell) use:
  ```
  dir Cert:\CurrentUser\My
  ```
  to find the newly created certificate's thumbprint

- export the certificate:
  ```
  Export-Certificate 
    -Cert 'Cert:\CurrentUser\My\<thumbprint>' 
    -FilePath 'certname.cer'
  ```

- if the certificate is self signed, deliver it to the users of the signed `.rdp`s, so that they can install it as a Trusted Root in their certificate store. 

  In an AD environment, you can deliver it via Group Policy

- add the certificate thumbprint to the `TrustedCertThumbprints` property of 
`HKCU:\Software\Policies\Microsoft\Windows NT\Terminal Services`.

  `TrustedCertThumbprints` is of type szString, and the value must be one or more thumbprints separated by commas

  You can use (for example) a `.reg`, or `.cmd` or `.ps1` to make this change on users's PCs.

  Note that in an AD environment, setting the above registry value is the same as setting the 
  ```
  User Configuration\ 
    Administrative Templates\ 
      Windows Components\ 
        Remote Desktop Services\ 
          Remote Desktop Connection Client\
            Specify SHA1 thumbprints of certificates representing trusted .rdp publishers
  ```
  Group Policy
  
Once the users have installed the certificate as a Trusted Root in the cert store, and defined it as trusted certificate throught the registry (or group policy), double clicking on a signed `.rdp` will start the Remote App without any security prompt.

