# MENGATASI ERROR 0x2746 Pada Koneksi SQL Server Aplikasi Pihak Ketiga (Laravel)

Pada kasus kali ini terjadi pada windows server 2012 dengan SQL Server 2008 R2

---

## Tahap Pertama

Pastikan SQL Server 2008 R2 sudah di update ke versi terbaru atau versi terakhir di SP3 dengan versi **10.50.6560** jika belum silahkan update terlebih dahulu, link download [versi 10.50.6560](https://www.microsoft.com/en-us/download/details.aspx?id=56415)


## Tahap Kedua

Cek Certifates - Local Computer dengan cara :

`Windows + R -> certlm.msc -> Personal -> Certifates`

Apabila kosong (akar masalah yang membuat error 0x2746) silahkan buat terlebih dahulu Certifates nya dengan menggunakan skrip dibawah ini : 

``` 
[Version]
Signature="$Windows NT$"

[NewRequest]
Subject = "CN=WIN-B2SDUQLC975" (WIN-B2xxx disesuaikan dengan nama hostname server)
KeySpec = 1
KeyLength = 2048
Exportable = TRUE
MachineKeySet = TRUE
ProviderName = "Microsoft RSA SChannel Cryptographic Provider"
ProviderType = 12
RequestType = Cert
HashAlgorithm = SHA256
KeyUsage = 0xa0

[EnhancedKeyUsageExtension]
OID=1.3.6.1.5.5.7.3.1

[Extensions]
2.5.29.17 = "{text}"
_continue_ = "DNS=WIN-B2SDUQLC975&" (WIN-B2xxx disesuaikan dengan nama hostname server)
_continue_ = "IPAddress=192.168.100.127&" (IPAddress disesuaikan dengan IP Server)

 ```
 Kemudian simpan dengan nama cert.inf dan jalankan perinta berikut di command prompt :
``` certrq -new C:\cert.inf C:\cert.cer ```

Setelah berhasil membuat Certifates silahkan cek kembali pada Certifates - Local Computer apakah Certifates sudah ada di folder Personal.

Cek sertifikat dengan command berikut pada Power Shell dan pastikan signature nya SHA-256 :
```
Get-ChildItem Cert:\LocalMachine\My | ForEach-Object {
  $_ | Select-Object Subject, Thumbprint,
    @{n='SigOID';e={$_.SignatureAlgorithm.Value}},
    @{n='KeySize';e={$_.PublicKey.Key.KeySize}},
    @{n='EKU';e={$_.EnhancedKeyUsageList.FriendlyName -join ','}}
} | Format-List
````
 
Pastikan SigOID = 1.2.840.113549.1.1.11 (ini sha256RSA) — kalau ...1.5 berarti masih SHA-1, KeySize = 2048, EKU = Server Authentication
```
Subject    : CN=WIN-B2SDUQLC975
Thumbprint : 2446BF2AB7A5C4089614965D72C9E6C56C42C1C3
SigOID     : 1.2.840.113549.1.1.11
KeySize    : 2048
EKU        : Server Authentication
 ```

## Tahap Ketiga
Beri akses private key ke service NT Service pada Certifates

`Klik kanan pada Certifates -> All Task -> Manage Private Keys -> Security -> Add NETWORK Service -> Permisions For System -> Read -> Apply/OK `

Setelahnya lakukan restart service MSSQLSERVER dengan command prompt
``` net stop MSSQLSERVER && net start MSSQLSERVER ```

dan langkah terakhir cek Certifates di registry dengan command berikut :
```reg query "HKLM\SOFTWARE\Microsoft\Microsoft SQL Server\MSSQL10_50.MSSQLSERVER\MSSQLServer\SuperSocketNetLib" /v Certificate ```

TARAA... sampai sini error 0x2746 sudah solved, 
Jika ingin menambahkan Certificate ke sql server maka lanjut ke tahap selanjutnya jika ingin pasang ssl di server (opsional)
