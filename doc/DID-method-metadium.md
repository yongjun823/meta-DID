Metadium DID Method Specification
=================
8th July 2019

# Table of Contents
1. [DID Method Name](#name)
2. [Method Specific Identifier](#identifier)
    1. [Example](#example1)
3. [DID Document](#document)
    1. [Example](#example2)
4. [JSON-LD Context Definition](#ld)
5. [CRUD Operation Definitions](#crud)
    1. [Create (Register)](#create)
    2. [Read (Resolve)](#read)
    3. [Update](#update)
    4. [Delete](#delete)
6. [Security](#security)
7. [Privacy](#privacy)
8. [References](#references)

Metadium is the next-generation identity system powered by blockchain technology. Metadium Decentralized Identifiers is a distributed identifier designed to provide a way for a community connected to the Metadium Ecosystem to uniquely identify an individual, organization, or digital device. The role of a Metadium DID is to provide a service that supports user-authentication and personal information verification. 

The Metadium DID method specification conforms to the requirements specified in 
the DID specification [**[1]**](https://w3c-ccg.github.io/did-spec/), currently published by the 
W3C Credentials Community Group. For more information about DIDs and DID method specifications, 
please see the DID Primer [**[2]**](https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2017/blob/master/topics-and-advance-readings/did-primer.md)

# DID Method Name <a name="name"></a>

The namestring that shall identify this DID method is: `meta`

A DID that uses this method MUST begin with the following prefix: `did:meta`. Per the DID specification, this string MUST be in lowercase. The remainder of the DID, after the prefix, is specified below.

# Method Specific Identifier <a name="identifier"></a>

The method specific identifier is composed of an optional Metadium network identifier with a `:` separator, followed by a Hex-encoded Metadium Identifier Number (MIN) (without a `0x` prefix).
```
meta-did = "did:meta:" + meta-specific-idstring
meta-specific-idstring = meta-network + ":" + MIN
meta-network = "mainnet" | "testnet"
meta-address = 40*HEXDIG
```
The MIN is case-insensitive, but it is recommended to use mixed-case checksum for address encoding (see [**[3]**](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md)).

## Example <a name="example1"></a>

Example `meta` DIDs:
```
did:meta:000000000000fd7022b4B4cAd5eF33723d2C549c85ad196b3db3
did:meta:mainnet:000000000000fd7022b4B4cAd5eF33723d2C549c85ad196b3db3
did:meta:testnet:000000000000fd7022b4B4cAd5eF33723d2C549c85ad196b3db3
```
# DID Document <a name="document"></a>

## Example <a name="example2"></a>
```
{
	"@context": "https://w3id.org/did/v1",
	"id": "did:meta:testnet:000000000000fd7022b4B4c...85ad196b3db3",
	"created": "2019-03-25T12:00:00Z",
	"updated": "2019-04-25T12:00:00Z",
	"publicKey": [{
		"id": "#MetaManagementKey#1",
		"type": "Secp256k1VerificationKey2018",
		"publicKeyHash": "e3FA89810623759d53361a297305c391c8280e66"
	       },
	       {
		"id": "#MetaServiceKey#1",
		"type": "Secp256k1VerificationKey2018",
		"publicKeyHash": "5A0b54D5dc17e0AadC383d2db43B0a0D3E029c4c"
	       }
	],
	"authentication": [{
		"id": "#MetaManagementKey#1",
		"type": "Secp256k1VerificationKey2018",
		"publicKeyHex": "034f355bdcb7cc0af728ef3cc...59ab0f0b704075871aa"
	       }
	],
	"service": [{
		"id": "#MetaManagementKey#1",
		"type": "IdentityHub",
		"serviceEndpoint": "https://identityhub.metadium.com"
	       }
	]
}
```
We use the ISO 8601 [**[4]**](https://www.iso.org/iso-8601-date-and-time-format.html) basic and extended notations for timestamp.
To make an public key hash from the public key, all we need to do is to apply Keccak-256 to the key and then take the last 20 bytes of the result. No Base58 or any other conversion.

# JSON-LD Context Definition <a name="ld"></a>

The `meta` method defines additional JSON-LD terms for the supported Metadium key types `MANAGEMENT`, and `SERVICE`.

The definition of the `meta` JSON-LD context is:
```
{
	"@context":
	{
		"METAManagementKey": "{TBA}",
		"METAServiceKey": "{TBA}"
	}
}
```
Note: Other type of keys, such as `recovery`, `resolver`, and `provider` of a DID Document may be supported in future versions of this specification.

# CRUD Operation Definitions <a name="crud"></a>

## Create (Register) <a name="create"></a>

In order to create a `meta` DID, a Metadium Identity Manager (MIM) smart contract, and Metadium Service Manager (MSM) smart contract must be deployed on Metadium blockchain.
MIM is compliant with the EIP1484 [**[5]**](https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1484.md) standard, and MSM is a default service key resolver.

`meta` DID creation is done by submitting a transaction to the MIM smart contract invoking the following method:
```
function createIdentity(address recoveryAddress, address[] memory providers, address[] memory resolvers) public returns (uint min);
```
This will generate the corresponding id-string (MIN) and assign control to the caller address. Identities are denominated by MINs, which are unique but otherwise uninformative.

## Read (Resolve) <a name="read"></a>

To construct a valid DID document from an `meta` DID, the following steps are performed:

1. Determine the Metadium network identifier ("mainnet", or "testnet"). If the DID contains no network identifier, then the default is "mainnet".
1. Invoke the `function getIdentity(uint min)` function to MIM for getting `MANAGEMENT` key.
1. For each returned key address, look up the associated key.
1. For each `MANAGEMENT` public key hash:
	1. Add a `publicKey` element of type `Secp256k1VerificationKey2018` and `MetaManagementKey` to the DID Document.
    1. Add an `authentication` element of type `Secp256k1VerificationKey2018`, referencing this `publicKey`.
1. Invoke the `function getKeys(uint min)` function to MSM for getting  for `SERVICE` key.
1. For each `SERVICE` public key hash:
	1. Add a `publicKey` element of type `Secp256k1VerificationKey2018` and `MetaServiceKey` to the DID Document.
	1. Add an `service` element of type  `Secp256k1VerificationKey2018`, referencing this `publicKey`.

Note: Service endpoints and other elements of a DID Document may be supported in future versions of this specification.

## Update <a name="update"></a>

The DID Document may be updated by invoking the relevant MSM smart contract functions as follows:
```
function addKey(address _key, uint min, bytes name)
function removeKey(address _key, uint min)
```
## Delete (Revoke) <a name="delete"></a>

Revoking the DID can be supported by executing a `destructIdentity` operation that is part of the MIM smart contract. This will remove the MIM and MSM's storage and code from the state, effectively marking the DID as revoked.
```
function destructIdentity(uint min)
```

# Security Considerations <a name="security"></a>

When a user creates and registers its own `meta` DID in the metadium blockchain, he (or she) can selectively register either recovery key or provider key. 
- Recovery key is a Metadium address (either an external account or smart contract) that can be used to recover lost Identities when you accidentally lose your private key. The recovery key must be set to a different value than the management key. It is safest to physically and logically allow a trusted third party, separate from the user, to archive the recovery key.
- The provider key is the Metadium address (external account or smart contract) that is authorized to be used on behalf of the management key. Since the provider key is allowed to operate in place of the management key, the provider key must be registered only when the user needs it, not at the time of distribution. Also, do not forget to delete the provider key when the proxy operation is complete.

# Privacy Considerations <a name="privacy"></a>

- The Metadium blockchain will have a claim/achievement structure in the next development phase. Claims are verifiable credentials, often consisting of a hash of personal information. GDPR protects pseudonymized data because of the "linkability" of an unreadable hash. Therefore, claims must be stored in a blockchain in such a way that personal information can not be inferred from hash-processed claims.

# References <a name="references"></a>
----------

 **[1]** https://w3c-ccg.github.io/did-spec/

 **[2]** https://github.com/WebOfTrustInfo/rebooting-the-web-of-trust-fall2017/blob/master/topics-and-advance-readings/did-primer.md

 **[3]** https://github.com/ethereum/EIPs/blob/master/EIPS/eip-55.md

 **[4]** https://www.iso.org/iso-8601-date-and-time-format.html

 **[5]** https://github.com/ethereum/EIPs/blob/master/EIPS/eip-1484.md
 
