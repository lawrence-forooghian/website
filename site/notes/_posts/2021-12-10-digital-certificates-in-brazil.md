---
title: Digital certificates in Brazil
---

# Digital certificates in Brazil

In Brazil, any _pessoa física_ (person) or _pessoa jurídica_ (legal entity, e.g. company) can get a digital certificate (and accompanying private key) that proves their identity.

This lets you:

- Sign documents (I don’t know _how_ but there’s a [signature verifier](https://verificador.iti.gov.br/verifier-2.7/) which mentions a standard called DOC-ICP-15), which according to [Medida Provisória nº 2.200-2, de 2001](http://www.planalto.gov.br/ccivil_03/mpv/antigas_2001/2200-2.htm) has legal validity in Brazil
- Sign emails
- Identify yourself to websites
- Increase the security level of your GOV.BR account (an example of “identify yourself to websites”, I suppose), which then gives you access to certain restricted parts of the Brazilian government websites, for example viewing previous years’ tax returns in [e-CAC](https://cav.receita.fazenda.gov.br/eCAC/publico/login.aspx).

## Who manages them?

Infra-Estrutura de Chaves Públicas Brasileira - [ICP-Brasil](https://www.gov.br/iti/pt-br/assuntos/icp-brasil). This seems to be part of the [Instituto Nacional de Tecnologia da Informação](https://www.gov.br/iti/pt-br).

They act as a CA. Their [root certificates are available on gov.br](https://www.gov.br/iti/pt-br/assuntos/repositorio/repositorio-ac-raiz), and don’t seem to come with macOS. (Some of their own websites, e.g. https://estrutura.iti.gov.br, use SSL certs with these roots, which I found interesting.)

You don’t purchase your certificate directly from them, but rather from one of many sub-CAs (some private, some public sector).

The certificates come in different levels (A1, A3, …), with different validity periods, capabilities, and storage requirements (e.g. A1 is a file on your computer whereas A3 is only provided on a smart card or USB token or "in the cloud" which I need to figure out the meaning of).

## How to get one?

[Pick a CA and go through the application process](https://www.gov.br/iti/pt-br/assuntos/certificado-digital/como-obter). You’ll probably need to go a real-life place with (Brazilian) government-issued ID to prove your identity. They’ll also take fingerprints and a photo (for reasons I don’t yet understand). I did mine through Prodesp and was able to book an appointment for the next day in a [Poupatempo](https://www.poupatempo.sp.gov.br) office. (Poupatempo being, I think, a state-run place to go and do various bureacracy things — I went to the one in Lapa and found it to be a surprisingly pleasant place to spend time, it's colourful and bright and airy.)

### Generating a certificate from Prodesp on macOS

I chose Prodesp (formerly Imprensa Oficial de São Paulo) because they seemed to be the cheapest option (R$100 for an A1 certificate). However, they do not officially support macOS. This is because they don’t vend a macOS version of the SafeSign IC software they they use for the emission of the certificate (even though the vendor supports macOS). And the software isn’t generic, it’s tied to a particular CA. I don’t know exactly what this software does, but presumably it generates a private key and a certificate signing request.

I used one of [Microsoft’s free Windows virtual machines](https://developer.microsoft.com/en-us/microsoft-edge/tools/vms/) to generate the cert (VirtualBox is a free virtual machine, which I used.)

At the end of the certificate generation process, SafeSign asks if you want to export the certificate and private key as a p12 file. You can then export this from the virtual machine (using Shared Folders in VirtualBox, for example) and import it using Keychain Utility on the Mac. You can then throw the virtual machine away.

## What does the cert look like?

I need to look in more detail. Looks like an X.509 cert. The Common Name is in the format "[my name]:[my [CPF number](https://en.wikipedia.org/wiki/CPF_number)]". On first glance, doesn’t seem to contain any additional information about me.

## Digital signatures from GOV.BR

There seems to be another type of digital signature (see [Assinatura Eletrônica do gov.br](https://www.gov.br/governodigital/pt-br/assinatura-eletronica)), which allows you to sign documents using your gov.br identity. According to the linked page, these are also legally valid. I tried downloading my cert from them, and it appears they use their own root certs, separate from those of ICP-Brasil. One to investigate more.
