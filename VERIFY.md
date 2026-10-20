# Verifying the files

Three independent levels. Level 1 is enough to detect corruption; levels 2–3 prove the files come from
the author.

Signing key: **Vincent Liu &lt;elsevier2012@hotmail.com&gt;**
Fingerprint: `DA1A 1AEF D8B2 C17C FF3F 1D1B 3905 0F50 7A4F EC17`
The same fingerprint is published in the Zenodo record [10.5281/zenodo.23480697](https://doi.org/10.5281/zenodo.23480697) and on
[GitHub](https://github.com/AlienMuse.gpg) (`https://github.com/AlienMuse.gpg`).

## Level 1 — checksums

Download the PDFs and `SHA256SUMS` into one folder.

Linux / macOS:

```sh
sha256sum -c SHA256SUMS --ignore-missing
```

Windows PowerShell:

```powershell
Get-Content SHA256SUMS | ForEach-Object {
  $hash, $name = $_ -split '  ', 2
  if (Test-Path $name) {
    $ok = (Get-FileHash $name -Algorithm SHA256).Hash -eq $hash.ToUpper()
    "{0}  {1}" -f ($(if ($ok) { 'OK  ' } else { 'FAIL' })), $name
  }
}
```

## Level 2 — signature

```sh
gpg --import VincentLiu-signing-key.asc          # from this repository or the release
gpg --verify SHA256SUMS.asc SHA256SUMS
```

The output must say `Good signature from "Vincent Liu <elsevier2012@hotmail.com>"` and show the fingerprint
above.

## Level 3 — timestamps and platform attestation
GitHub releases of this repository are immutable and carry a GitHub-signed release attestation:

```sh
gh release verify v1.0.0 -R AlienMuse/grothendieckian-machine
gh release verify-asset v1.0.0 GAI-MUA-v1.0.0-Vol1-Foundations.pdf -R AlienMuse/grothendieckian-machine
```

`SHA256SUMS.tsr` is an RFC 3161 timestamp token over `SHA256SUMS` from FreeTSA:

```sh
openssl ts -reply -in SHA256SUMS.tsr -text          # shows the time
openssl ts -verify -data SHA256SUMS -in SHA256SUMS.tsr \
    -CAfile timestamps/freetsa-cacert.pem -untrusted timestamps/freetsa-tsa.crt
```

A second, independent token over the same file comes from DigiCert. Its certificate chain up to the public
*DigiCert Trusted Root G4* is in `timestamps/digicert-chain.pem`; verify against your system's CA bundle:

```sh
openssl ts -verify -data SHA256SUMS -in timestamps/SHA256SUMS.digicert.tsr \
    -CAfile /etc/ssl/certs/ca-certificates.crt -untrusted timestamps/digicert-chain.pem
```

`SHA256SUMS.ots` anchors the same file in the Bitcoin blockchain (<https://opentimestamps.org>):

```sh
ots verify SHA256SUMS.ots
```

## Checksums of this release

```text
27491d9a3b431bc88a38ba28e28c58b5e7c1bde6fe0e36ba6ea1c6afe33ab3b5  GAI-MUA-v1.0.0-Complete.pdf
41957f501c2f448f8d13b36bd5060a72326842ca56b837233227c5602c8cb8a9  GAI-MUA-v1.0.0-Vol1-Foundations.pdf
4380e6c81a231dddce8820f920f1c1dd5bf09a2892ec8169692f4941f145e58a  GAI-MUA-v1.0.0-Vol2-Modern-Synthesis.pdf
bb0f03c579a30a0d42161a0b4749af6afb27dcd909a77c99a1d789bde0f3dbb3  GAI-MUA-v1.0.0-Vol3-Architecture.pdf
ce545f3c8bbc9b63e601dcae614a85528858192da5bc9ecf9ca89506e5e459b1  GAI-MUA-v1.0.0-Vol4-Algorithms.pdf
00ab460cd8eb26bae5aed54f78a6569fc5434a96e54b502c0e9ccfacd9d74e50  GAI-MUA-v1.0.0-Vol5-Modular-Machine.pdf
ccda27d932bec5f881ebf5776efe405ef7a784a49b95c34b9fada3e9f2f7bf34  GAI-MUA-v1.0.0-Vol6-Self-Evolution.pdf
52ee0a3f7633b9f4d55656ca1a7f0765496e7b01b872e680d47f4b9ac9a2242e  GAI-MUA-v1.0.0-Preview.pdf
```
