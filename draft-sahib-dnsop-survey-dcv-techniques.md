---
title: "A Survey of Domain Control Validation Techniques using DNS"
abbrev: "Survey of Domain Control Validation"
docname: draft-sahib-dnsop-survey-dcv-techniques-latest
category: info

ipr: trust200902
keyword: Internet-Draft

stand_alone: yes
stream: IETF
smart_quotes: no
pi: [toc, sortrefs, symrefs]

author:
 -
    ins: S. Sahib
    name: Shivan Sahib
    organization: Brave Software
    email: shivankaulsahib@gmail.com

 -
    ins: S. Huque
    name: Shumon Huque
    organization: Salesforce
    email: shuque@gmail.com

 -
    ins: E. Nygren
    name: Erik Nygren
    organization: Akamai Technologies
    email: erik+ietf@nygren.org

 -
    ins: T. Wicinski
    name: Tim Wicinski
    organization: Cox Communications
    email: tjw.ietf@gmail.com

normative:
  RFC1034:
  RFC1464:

informative:
    RFC8555:
    I-D.draft-ietf-dnsop-domain-verification-techniques:
    LETSENCRYPT-DNS01:
        title: "Challenge Types"
        author:
          - ins: Internet Security Research Group (ISRG)
        target: https://letsencrypt.org/docs/challenge-types/#dns-01-challenge

    LETSENCRYPT-90DAYS:
        title: "Why ninety-day lifetimes for certificates?"
        date: 2015
        author:
          - ins: J. Aas
        target: https://letsencrypt.org/2015/11/09/why-90-days.html

    GOOGLE-WORKSPACE-TXT:
        title: "TXT record values"
        author:
          - ins: Google
        target: https://support.google.com/a/answer/2716802

    GOOGLE-WORKSPACE-CNAME:
        title: "CNAME record values"
        author:
          - ins: Google
        target: https://support.google.com/a/answer/112038

    ATPROTO-HANDLE:
        title: "Handle"
        author:
          - ins: Bluesky
        target: https://atproto.com/specs/handle#dns-txt-method

    GITHUB-ORG-VERIFY:
        title: "Verifying your organization's domain"
        author:
          - ins: GitHub
        target: https://docs.github.com/en/github/setting-up-and-managing-organizations-and-teams/verifying-your-organizations-domain

    PSL:
        title: "Public Suffix List"
        author:
          - ins: Mozilla Foundation
        target: https://publicsuffix.org/

    DOCUSIGN-CNAME:
        title: "Verify a domain with a CNAME record"
        author:
          - ins: DocuSign
        target: https://support.docusign.com/s/document-item?bundleId=rrf1583359212854&topicId=gso1583359141256_1.html

    ACM-CNAME:
        title: "AWS Certificate Manager DNS validation"
        author:
          - ins: Amazon Web Services
        target: https://docs.aws.amazon.com/acm/latest/userguide/dns-validation.html

    AKAMAI-DCV:
        title: "Add a hostname with a default certificate"
        author:
          - ins: Akamai Technologies
        target: https://techdocs.akamai.com/property-mgr/docs/add-hn-with-default-cert-la

    DIGICERT-DCV:
        title: "CertCentral change log"
        date: 2025
        author:
          - ins: DigiCert
        target: https://docs.digicert.com/en/whats-new/change-log/certcentral-change-log.html

    FASTLY-TLS:
        title: "Setting up TLS with certificates Fastly manages"
        author:
          - ins: Fastly
        target: https://docs.fastly.com/en/guides/setting-up-tls-with-certificates-fastly-manages

    CLOUDFLARE-DCV:
        title: "Delegated DCV"
        author:
          - ins: Cloudflare
        target: https://developers.cloudflare.com/ssl/edge-certificates/changing-dcv-method/methods/delegated-dcv/

    F5-ACME:
        title: "What is an ACME challenge and how should I enter these records in my DNS zone?"
        author:
          - ins: F5, Inc.
        target: https://f5cloud.zendesk.com/hc/en-us/articles/24624288087959-What-is-an-ACME-challenge-and-how-should-I-enter-these-records-in-my-DNS-zone

    GOOGLE-CERT-MANAGER:
        title: "Deploy a global Google-managed certificate with DNS authorization"
        author:
          - ins: Google
        target: https://cloud.google.com/certificate-manager/docs/deploy-google-managed-dns-auth

    CERTIFY-DNS:
        title: "Certify DNS"
        author:
          - ins: Certify The Web
        target: https://docs.certifytheweb.com/docs/dns/providers/certifydns/

    ACME-DNS:
        title: "acme-dns"
        author:
          - ins: J. Hoikkala
        target: https://github.com/acme-dns/acme-dns


--- abstract

{{I-D.draft-ietf-dnsop-domain-verification-techniques}} describes best practices for using the DNS to verify ownership or control of a domain, a process generally referred to as "Domain Control Validation". This document is a companion survey that catalogs the DNS-based Domain Control Validation techniques in use today across a range of Application Service Providers and protocols.


--- middle

# Introduction

Application Service Providers often need domain owners to prove control of a DNS domain before providing a service, such as a Certification Authority (CA) issuing a TLS certificate. DNS-based methods typically ask the requester to publish one or more DNS records, with content and placement specified by the Application Service Provider, that the provider can then query for.

This document surveys such DNS-based Domain Control Validation techniques in use today, cataloging the record types (primarily TXT and CNAME), naming conventions, and token placement drawn from the publicly documented behavior of a range of Application Service Providers and protocols. It makes no recommendations; see {{I-D.draft-ietf-dnsop-domain-verification-techniques}} for best practices.

# Conventions and Definitions

This document uses the terminology defined in the "Conventions and Definitions" section of {{I-D.draft-ietf-dnsop-domain-verification-techniques}}, including "Application Service Provider", "DNS Administrator", "User", "Unique Token", and "Validation Record".

# TXT based

A TXT record is usually the default option for domain control validation. The Application Service Provider asks the User to add a DNS TXT record (perhaps through their domain host or DNS provider) at the domain with a certain value. Then the Application Service Provider does a DNS TXT query for the domain being verified and checks that the correct value is present. For example, this is what a DNS TXT record could look like for an Application Service Provider `Service`:

    example.com.   IN   TXT   "237943648324687364"

Here, the value "237943648324687364" serves as the randomly-generated TXT value being added to prove ownership of the domain to `Service` Application Service Provider. Note that in this construction Application Service Provider `Service` would have to query for all TXT records at "example.com" to get the Validation Record. Although the original DNS protocol specifications did not associate any semantics with the DNS TXT record, {{RFC1464}} describes how to use them to store attributes in the form of ASCII text key-value pairs for a particular domain. In practice, there is wide variation in the content of DNS TXT records used for domain control validation, and they often do not follow the key-value pair model. Even so, the RDATA {{RFC1034}} portion of the DNS TXT record has to contain the value being used to verify the domain. The value is usually a Unique Token in order to guarantee that the entity who requested that the domain be verified (i.e., the person managing the account at Application Service Provider `Service`) is the one who has (direct or delegated) access to DNS records for the domain. After a TXT record has been added, the Application Service Provider will usually take some time to verify that the DNS TXT record with the expected token exists for the domain. The generated token typically expires in a few days.

Some Application Service Providers use a prefix of `_PROVIDER_NAME-challenge` in the Name field of the TXT record challenge. For ACME, the full Host is `_acme-challenge.<YOUR_DOMAIN>`. Such patterns are useful for doing targeted domain control validation. The ACME protocol ({{RFC8555}}) has a challenge type `DNS-01` that lets a User prove domain ownership. In this challenge, an implementing CA asks you to create a TXT record with a randomly-generated token at `_acme-challenge.<YOUR_DOMAIN>`:

    _acme-challenge.example.com.  IN  TXT "cE3A8qQpEzAIYq-T9DWNdLJ1_YRXamdxcjGTbzrOH5L"

{{RFC8555}} (Section 8.4) places requirements on the Unique Token.

## Let's Encrypt

The ACME example is implemented by Let's Encrypt {{LETSENCRYPT-DNS01}}.

## Google Workspace

Google Workspace {{GOOGLE-WORKSPACE-TXT}} asks the User to sign in with their administrative account and obtain their token as part of the setup process for Google Workspace. The verification token is a 68-character string that begins with "google-site-verification=", followed by 43 characters. Google recommends a TTL of 3600 seconds. The owner name of the TXT record is the domain or subdomain name being verified.

## The AT Protocol

The Authenticated Transfer (AT) Protocol supports DNS TXT records for resolving social media "handles" (human-readable identifiers) to the User's persistent account identifier {{ATPROTO-HANDLE}}. For example, this is how the handle `bsky.app` would be resolved:

    _atproto.bsky.app.  IN  TXT "did=did:plc:z72i7hdynmk6r22z27h6tvur"

## GitHub

To verify domains for organizations, GitHub asks the User to create a DNS TXT record under `_github-challenge-ORGANIZATION.<YOUR_DOMAIN>`, where ORGANIZATION stands for the GitHub organization name. The RDATA value for the provided TXT record is a string that expires in 7 days {{GITHUB-ORG-VERIFY}}.

## Public Suffix List

The Public Suffix List {{PSL}} asks owners of private domains to authenticate by creating a TXT record containing the pull request URL for adding the domain to the Public Suffix List. For example, to authenticate "example.com" submitted under pull request 100, a requestor would add:

    _psl.example.com.  IN TXT "https://github.com/publicsuffix/list/pull/100"

# CNAME based

## CNAME for Domain Control Validation

### DocuSign

DocuSign {{DOCUSIGN-CNAME}} asks the User to add a CNAME record with the "Host Name" set to be a 32-digit random value pointing to `verifydomain.docusign.net.`.

### Google Workspace CNAME

Google Workspace {{GOOGLE-WORKSPACE-CNAME}} lets you specify a CNAME record for verifying domain ownership. The User gets a unique 12-character string that is added as "Host", with TTL 3600 (or default) and Destination an 86-character string beginning with "gv-" and ending with ".domainverify.googlehosted.com.".

## Delegated Domain Control Validation

### Content Delivery Networks (CDNs)

In order to be issued a TLS cert from a Certification Authority like Let's Encrypt, the requester needs to prove that they control the domain. Often this is done via the DNS-01 challenge {{LETSENCRYPT-DNS01}}. Let's Encrypt only issues certs with a 90 day validity period for security reasons {{LETSENCRYPT-90DAYS}}. This means that after 90 days, the DNS-01 challenge has to be re-done and the Unique Token has to be replaced with a new one. Doing this manually is error-prone.

Content Delivery Networks offer to automate this process using a CNAME record in the User's DNS that points to the Validation Record in the CDN's zone.

### AWS Certificate Manager (ACM)

AWS Certificate Manager {{ACM-CNAME}} allows delegated domain control validation. The record name for the CNAME looks like:

    _<random-token1>.example.com.  IN   CNAME _<random-token2>.acm-validations.aws.

The CNAME points to:

    _<random-token2>.acm-validations.aws.  IN   TXT "<random-token3>"

Here, the random tokens are used for the following:

* `<random-token1>`: Unique sub-domain, so there are no clashes when looking up the Validation Record.
* `<random-token2>`: Proves to ACM that the requester controls the DNS for the requested domain at the time the CNAME is created.
* `<random-token3>`: The actual token being verified.

Note that if there are more than 5 CNAMEs being chained, then this method does not work.

### Akamai

Akamai {{AKAMAI-DCV}} uses a CNAME record:

    _acme-challenge.domain.com. IN CNAME validation.validate-akdv.net.

### DigiCert

DigiCert is switching to use a static prefix `_dnsauth` CNAME record configuration {{DIGICERT-DCV}}:

    _dnsauth.example.com. IN CNAME [random_value].dcv.digicert.com.

### Fastly

Fastly {{FASTLY-TLS}} uses a CNAME record:

    _acme-challenge.domain.com. IN CNAME token.fastly-validations.com.

### Cloudflare

Cloudflare {{CLOUDFLARE-DCV}} uses a CNAME record at `_acme-challenge.domain.com.` pointing to a Cloudflare validation target, for partial DNS setups.

### F5 Distributed Cloud

Customers create a CNAME record for

    _acme-challenge.demo.f5lab.com.

with the value provided by F5 Distributed Cloud {{F5-ACME}}.

### Google Cloud Certificate Manager

Google's Certificate Manager {{GOOGLE-CERT-MANAGER}} uses DNS authorization where customers add CNAME records like

    _acme-challenge.myorg.example.com.

pointing to Google's validation domains for certificate provisioning.

### Certify DNS

Certify DNS {{CERTIFY-DNS}} is a cloud hosted version of the acme-dns protocol that uses CNAME delegation of ACME challenge TXT records to a dedicated challenge response service.

### acme-dns (open source)

acme-dns {{ACME-DNS}} is an open source limited DNS server with a RESTful HTTP API where customers create CNAME records like

    _acme-challenge.domainiwantcertfor.tld. IN  CNAME a097455b-52cc-4569-90c8-7a4b97c6eba8.auth.example.org.

# Security Considerations

This document is a survey of existing Domain Control Validation techniques and does not itself define a new protocol. The techniques surveyed here have varying security properties; some of the operational and security considerations that apply to them, including the construction of tokens and the placement of Validation Records, are discussed in {{I-D.draft-ietf-dnsop-domain-verification-techniques}}.

# IANA Considerations

This document has no IANA actions.


--- back

# Acknowledgments
{:numbered="false"}

This survey is derived from material originally developed as part of the Domain Control Validation using DNS work in the DNS Operations Working Group. Thanks to the contributors and reviewers of that document.
