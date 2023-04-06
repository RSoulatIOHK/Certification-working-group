---
CIP: 123
Title: On-chain Certification Metadata Standard
Category: Metadata
Status: Proposed
Authors:
    - Romain Soulat <romain.soulat@iohk.io>
    - Nicolas Jeannerod <nicolas.jeannerod@tweag.io>
    - Mathieu Montin <mathieu.montin@tweag.io>
    - Ali Modiri <mixaxim@gimbalabs.io>
Implementors: []
Discussions:
    - https://github.com/cardano-foundation/cips/pulls/??
    - https://github.com/input-output-hk/Certification-working-group/pull/18
Created: 2023-02-23
License: CC-BY-4.0
---


## Abstract
Certification is the process of providing various kinds of assurance of DApps through the provision of verifiable, trustworthy metadata linking to relevant evidence of assurance, such as audit reports, test run data, formal verification proofs etc.

Currently, certification does not have a standardised way to be stored and presented to all stakeholders (DApp developer, DApp stores, end-users, auditors, ...) in an immutable, persistent and verifiable way.

This CIP descibes a standardised method for certificates to be published and stored on-chain and for stakeholders to be able to verify the different claims of this certificate.

This proposal describes how certification metadata can be designed for DApp registration. It first describes the requirements for DApp certification, and then sets out the options for its inclusion in CIP-72.



## Motivation: why is this CIP necessary?

It is expected that evidence of various kinds of assurance of DApps is recorded in an immutable and verifiable way on the Cardano blockchain; these forms of assurance are: automated testing, including property-based testing (level 1), audit (level 2) and formal verification (level 3).

The metadata should be discoverable by all certification stakeholders, including end-users, DApp developers, and ecosystem components, such as light wallets and DApp stores. Information should be indexable by certification issuer, DApp developer, DApp and DApp version.

### Use-cases and Stakeholders
DApp developers will seek certification from one of the issuers, according to their desired level of certification and will refer to the certificate on several platforms (e.g. on their website or in their DApp registration on a DApp store).

Certification is to be provided by certification issuers including: testing services (level 1), auditors (level 2), and verification services (level 3). Certification issuers will issue these certificates referring to a particular version of a DApp that has succesfully gone through a certification process. 

DApp stores and light wallets will be able to pull DApps information from DApps registration or chain exploring and will link a DApp to a corresponding set of certificates.

End-user will interact with a DApp through a wallet and will be able to check the different certificates obtained by the DApp.


## Specification

### Definitions
- **anchor** - A hash written on-chain that can be used to verify the integrity (by way of a cryptographic hash) of the data that is found off-chain.
- **DApp** - A decentralised application that is described by the combination of metadata, certificate and a set of scripts.
- **dApp Store** - A dApp aggregator application which follows on-chain data looking for and verifying dApp metadata claims, serving their users linked dApp metadata.
- **Certification issuers** - A company that issues certification certificates on-chain.

### Certification issuers
Certification issuers will broadcast on-chain certificates that will represent th level of certification reached and present evidence of the work done.
Certification issuers will sign the certificate to attest that they have done the work and prevent certificate forgeries.

### Suggested validations
- `integrity`: The DApp's metadata off-chain shall match the metadata **anchored** on-chain.
- `trust`: The DApp's certification metadata transaction shall be signed by a trusted certfication issuer. This means a list of wallets of certification issuers needs to be known. It will then be up to the DApp store to decide which certification issuers are really trusted and publish a list of their own trusted certification issuers.

### Certificate JSON Schema

```json
{
    "$schema": "https://json-schema.org/draft/2019-09/schema" 
    "$id": "", 
    "$title": "Certification Certificate", 
    "type": "object", 
    "properties": {
      "subject": {
        "type": "string",
        "description": "Can be anything. Subject of the certification. It should match the registration metadata subject.",
      },
      "rootHash": {
        "type": "string",
        "description": "blake2b hash of the off-chain certification metadata linked in the metadata field."
      },
      "metadata": {
        "type": "array",
        "description": "Array of links that points to the metadata json file."
        "items": [
          {
            "type": "string"
          }
        ]
      },
      "schemaVersion": {
        "type": "integer",
        "description": "Used to describe the json schema version of the on-chain metadata."
      }
      "type": {
        "type": "object",
        "description": "Describes the certification certificate type.", 
        "properties": {
          "action": {
            "type": "string",
            "description": "Describes the action this certification certificate is claiming. For the moment, CERTIFY shall be used to claim a certification that meets the Certfication Standard. AUDIT shall be used to publish on-chain an audit that may not meet the Certification standard"
          },
          "certificationLevel": {
            "type": "integer",
            "description": "Integer between 1 and 3 to describe the level of certification this certificate refers to when using a CERTIFY action. Integer of 0 for an audit when using an AUDIT action."
          },
          "certificateIssuer": {
            "type": "string",
            "description": "Certification issuer name."
          }
        },
        "required": ["action", "certificationLevel", "certificateIssuer"]
      },  
    "required": ["subject", "rootHash","metadata", "schemaVersion", "type"]
}
```

### On-chain DApp Certification Certificate
```json
{
    "subject": "d684512ccb313191dd08563fd8d737312f7f104a70d9c72018f6b0621ea738c5b8213c8365b980f2d8c48d5fbb2ec3ce642725a20351dbff9861ce9695ac5db8", 
    "rootHash": "f08ccc1ee08d034d8317d1d84cab76d3cac48a8466ca9e54a291bb998c49a1732e93280bf04a11293c73195affe4fcaa41f7b27c067396f97f701dd96f72665e",
    "metadata": [
        "ipfs://abcdefghijklmnopqrstuvwxyz0123456789",
        "https://example.com/metadata.json"
    ],

    "schemaVersion": "1.0",
    "type": {
        "action": "CERTIFY",
        "certificationLevel": "1",
        "certificateIssuer": "Example LLC"
    }
}
```

### Properties

`subject`: Identifier of the claim subject (i.e dApp). A UTF-8 encoded string. 

`type`: The type of the claim. This is a JSON object that contains the following properties:

- `action`: The action that the certificate is asserting. It can take the following values:
    - `CERTIFY`: The certificate is asserting that the dApp is being certified.
    - `AUDIT`: The certificate is asserting that an audit has been performed for this DApp. `AUDIT` shall come with a certification level `certificationLevel` of `0`.

    - `certificationLevel`: The certification level is an integer between 0 and 3. 0 is reserved for audits and reports that are not compliant with the certification standards. 1, 2, and 3 refers to the certification certificates referring to the three level of certifications as defined by the certification working group.
    - `certificationIssuer`: The name of the certification issuer.



`rootHash`: The hash of the metadata entire tree object. This hash is used by clients to verify the integrity of the metadata tree object. When reading a metadata tree object, the client should calculate the hash of the object and compare it with the rootHash property. If the two hashes don't match, the client should discard the object. The metadata tree object is a JSON object that contains the dApp's metadata. The metadata tree object is described in the next section.

This hash is calculated by taking the entire metadata tree object, ordering the keys in the object alphanumerically, and then hashing the resulting JSON string using the **blake2b-256** hashing algorithm. The hash is encoded as a hex string.

`schemaVersion`: A versioning number for the json schema of the on-chain metadata. This current description shall be refered as `schemaVersion: 1`.

`metadata`: An array of links to the dApp's metadata. The metadata is a JSON object that contains the dApp's metadata in accordance with [CIP 26](https://cips.cardano.org/cips/cip26/)

<!-- `signature`: The signature of the certificate. The signature is done over the blake2b-256 hash of `rootHash`. Any stakeholder can use the public key to verify who signed the certificate and possibly compare it with a list of known public keys to check the identity of the certificate issuer. -->

Values for all fields shall be shorter than 64 characters to be able to be included on-chain.

### Metadata Label
When submitting the transaction metadata pick the following value for `transaction_metadatum_label`:

- `1304`: DApp Certification

### Off-chain Metadata Format
The Dapp Certification certificate is complemented by off-chain metadata that can be fetched from off-chain metadata servers.

### Properties

`subject`, a mandatory string, Identifier of the claim subject (i.e dApp). A UTF-8 encoded string.

`schemaVersion`, a mandatory string, used as a versioning number for the Json schema of the on-chain metadata. This current description shall be refered as `schemaVersion: 1`.

`certificateIssuer`, a mandatory object used to describe the certification issuer, that contains the following fields:
- `name`, a mandatory string, the name of the certification issuer
- `logo`, a string, a link to the logo of the certification issuer. The logo could be self-hosted in order to allow updates on the logo design.
- `social`, an object used to list all the different social contacts of the Certificate Issuer:
   - `twitter`, a string, the twitter handle of the Certificate Issuer
   - `github`, a string, the github handle of the Certificate Issuer
   - `contact`, a mandatory string, the contact email of the Certificate Issuer
   - `website`, a mandatory string, a link to the Certificate Issuer's website
   - `discord`, a string, a link to the Certificate Issuer's Discord server

`report`, an object that contains:
- `reportURLs`, an array of URLs, is a field where each link points to the same actual certification report for anyone to read. This ensures transparency in the findings, what was and was not considered in the certification process..
- `reportHash`, a string, is a field that is the blake2b-256 hash of the report pointed by the links in `reportURLs`. 

`summary`, a string, that contains the summary of the report from the certification issuer.

`disclaimer`, a string, that contains the legal disclaimer from the certification issuer.

`script` an array of objects that represents the whole DApp's on-chain script. Each object is comprised of:
- `fullScriptHash`, a string, is the prefix and script hash or script hash+staking key
- `scriptHash`, a string, is the script hash or script hash+staking key
- `contractAddress`, a string, the script address


The off-chain metadata should follow the following schema and should then be reformatted according to [RFC 8785](https://www.rfc-editor.org/rfc/rfc8785) on canonical JSON Scheme:

```json
{
  "$schema": "http://json-schema.org/draft-04/schema#",
  "type": "object",
  "properties": {
    "subject": {
      "type": "string"
      "description": "Can be anything. Subject of the certification. It should match the registration metadata subject and the on-chain metadata.",
    },
    "schemaVersion": {
      "type": "string",
      "description": "Used to describe the json schema version of the off-chain metadata."

    },
    "certificationLevel": {
      "type": "integer",
      "description": "Describes the action this certification certificate is claiming. For the moment, CERTIFY shall be used to claim a certification that meets the Certfication Standard. AUDIT shall be used to publish on-chain an audit that may not meet the Certification standard. Shall match the on-chain certificationLevel"
    },
    "certificateIssuer": {
      "type": "object",
      "properties": {
        "name": {
           "type": "string",
           "description": "Name of the Certificate Issuer."
        },
        "logo": {
            "type": "string"
            "description": "URL to the logo of the Certificate Issuer."
        },
        "social": {
          "type": "object",
          "properties": {
            "twitter": {
              "type": "string",
              "description": "Twitter handle of the Certificate Issuer."
            },
            "github": {
              "type": "string",
              "description": "GitHub handle of the Certificate Issuer."
            },
            "contact": {
              "type": "string",
              "description": "Contact email of the Certification Issuer."
            },
            "website": {
              "type": "string",
              "description": "URL to the website of the Certificate Issuer."
            },
            "discord": {
                "type": "string",
                "description": "URL to the Discord server of the Certificate Issuer."
            }
          },
          "description": "Social contacts of the Certificate Issuer.",
          "required": [
              "website",
              "contact"
          ]
        }
      },
      "required": [
          "name",
          "social"
      ]
    },
    "report": {
      "type": "object",
      "properties": {
        "reportURLs": {
          "type": "array",
          "items": [
              {
                "type": "string",
                "description": "A URL to the report of the certification/audit."
              }
          ]
        },
        "reportHash": {
          "type": "string",
          "description": "Hash of the report pointed by the links in reportURLs"
        }
      },
      "required": [
          "reportURLs",
          "reportHash"
      ]
    },
    "summary": {
      "type": "string",
      "description": "A string of the summary of the certification/audit. "
    },
    "disclaimer": {
      "type": "string",
      "description": "A string of the legal disclaimer from the Certificate Issuer"
    },
    "scripts": {
      "type": "array",
      "items": [
        {
            "smartContractInfo": {
                "type": "object",
                "properties": {
                    "era": {
                        "type": "string",
                        "description": "Describe which era this script was written for."

                    },
                    "compiler": {
                        "type": "string",
                        "description": "Compiler used for this script."
                    },
                    "compilerVersion": {
                        "type": "string",
                        "description": "Compiler version used for this script."
                    },
                    "optimizer": {
                        "type": "string",
                        "description": "Post compilation optimizer used for this script."
                    },
                    "optimizerVersion": {
                        "type": "string",
                        "description": "Post compilation optimizer version used for this script."
                    },
                    "progLang": {
                        "type": "string",
                        "description": "Programming language used for this script"
                    },
                    "repository": {
                        "type": "string",
                        "description": "URL to the repository of the script"
                    }
                }
            }
            "fullScriptHash": {
              "type": "string",
              "description": "Prefix and script hash or script hash+staking key"
            },
            "scriptHash": {
              "type": "string",
              "description": "Script hash or script hash+staking key"
            },
            "contractAddress": {
              "type": "string",
              "description": "Script on-chain address"
            }
            "required": [
                "fullScriptHash",
                "scriptHash",
                "contractAddress"
              ]
        }
      ]
    }
  },

  "required": [
    "subject",
    "schemaVersion",
    "certificationLevel",
    "certificateIssuer",
    "report",
    "summary",
    "disclaimer",
    "scripts"
  ]
}
```

### Example

```json
{
  "subject": "d684512ccb313191dd08563fd8d737312f7f104a70d9c72018f6b0621ea738c5b8213c8365b980f2d8c48d5fbb2ec3ce642725a20351dbff9861ce9695ac5db8",
  "schemaVersion": "1.0.0",
  "certificationLevel": 1,
  "certificateIssuer": {
    "name": "Audit House LLC",
    "logo": "https://www.example.com/media/logo.svg"
    "social": {
        "contact": "contact@example.com",
        "link": "https://example.com",
        "twitter": "twiterHandle",
        "github": "githubHandle",
        "website": "https://www.example.com"
    }
  },
  "report": {
    "reportURLs": [
        "https://example.io/certificate.pdf",
        "ipfs://bafybeiemxfal3jsgpdr4cjr3oz3evfyavhwq"
        ],
    "reportHash": "c6bb42780a9c57a54220c856c1e947539bd15eeddfcbf3c0ddd6230e53db5fdd"
  },
  "summary": "This is the summary of the report."
  "disclaimer": "This is the legal disclaimer from the report",
  "scripts": [
        {
          "smartContractInfo": {
            "era": "basho",
            "compiler": "plutusTx",
            "compilerVersion": "1.3.0",
            "optimizer": "plutonomy",
            "optimizerVersion": "v0.20220721",
            "progLang": "plutus-v2",
            "repository": "https://github.com/DAppDev/TestDApp/"
          },
          "fullScriptHash": "711dcb4e80d7cd0ed384da5375661cb1e074e7feebd73eea236cd68192",
          "scriptHash": "1dcb4e80d7cd0ed384da5375661cb1e074e7feebd73eea236cd68192",
          "contractAddress": "addr1wywukn5q6lxsa5uymffh2esuk8s8fel7a0tna63rdntgrysv0f3ms"
        },
        {
          "smartContractInfo": {
            "era": "basho",
            "compiler": "plutusTx",
            "compilerVersion": "1.3.0",
            "optimizer": "plutonomy",
            "optimizerVersion": "v0.20220721",
            "progLang": "plutus-v2",
            "repository": "https://github.com/DAppDev/TestDApp/"
          },
          "fullScriptHash": "384da5375661cb1e07713eea236cd681921dcb4e80d7cd0ed4e7feebd7",
          "scriptHash": "6cd68191dcb4e80d7c5661cb1e074e7feebd73eea232d0ed384da537",
          "contractAddress": "addr1wywukn5q6lrdntgrysv0f7a0tna63ymffh23ms5uxsaesuk8s8fel"
        }
  ]
}
```

<!-- [TO DISCUSS: Verification tools, how do we represent that? with versions, compilation options] -->

The metadata should be discoverable by all certification stakeholders, including end-users, DApp developers, and ecosystem components, such as light wallets and DApp stores. Information should be indexable by certification issuer, DApp developer, DApp and DApp version.

It should be possible for there to be multiple versions of metadata published by the same certification issuer for the same version of a DApp, particularly when a (minor) update of the evidence is necessary. In such a case it should be possible to identify the most recent version of the report for that DApp version. It will also be the case that the same DApp may have certification metadata provided by multiple different certification issuers.

It should be possible for wallets to identify to users the certification status of a DApp when they are signing a transaction that is being submitted to a deployed DApp.

### Custom fields
Certification issuers should be free to add additional fields to fit some additional needs.


### Certification issuers
| Certification issuer | URL                | Contact email        | Certification levels | Cardano address |
|----------------------|-----               |---------------       |----------------------|------------   |
| Example Ltd.         | http://example.com | contact@example.com  | 1,2                  | EXAMPLEADDRESS  |
 
## Rationale: how does this CIP achieve its goals?
An on-chain solution is preferred as it allows for it to be checkable by any stakeholder and immutable.

Certificate are issued by certification issuers that sign the certificate to prevent certificate forgeries.
This design allows for anyone to issue certificates as long as they sign it but stakeholders are then free to maintain a list of the trusted certification issuers.

These certificates are self-standing and can be presented as-is by any stakeholder.

This proposal does not affect any backward compatibility of existing solution but is based on the work done for CIP72 on DApp registration and Discovery. It is also linked to CIP52 on Cardano audit best practices guidelines.

### Other designs considered
**Updates to registration entries**
This would have required DApp owner or certification issuers to add certificates to every registration making it harder to maintain a shared state between all stakeholders.
The chosen design requires to follow the chain to discover the certificates which should be expected from stakeholders.


## Path to Active

### Acceptance Criteria

Certificates are being issued under this form by multiple DApps auditors and certification companies.

Certificates are being displayed by multiple DApps stores or aggregators which uses this format.

### Implementation Plan
 - [x] This CIP will be discussed and agreed by the Cardano Certification Working Group where multiple auditors are being represented. This will ensure that certification issuer have agreed on the content and are ready to issue certificates under this format.

 - [x] This CIP will be presented to the IOG Lace team and Cardano Fans team which will display certificates for DApps. This is to ensure that the format contains all the necessary information for a DApp store or an aggregator to correctly link and display a certificate from the on-chain and off-chain metadata.

## Copyright
This CIP is licensed under [CC-BY-4.0].
[CC-BY-4.0]: https://creativecommons.org/licenses/by/4.0/legalcode
