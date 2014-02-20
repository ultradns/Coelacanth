# DNS JSON Spec
The goal of this project is to openly define a DNS JSON Specification that will facilitate interoperability between DNS Providers.

[UltraDNS](http://www.neustar.biz/enterprise/dns-services) is proposing the following format, but we would like to receive feedback from the community and define a format that anyone can implement.

## Definitions
- Fully Qualified Domain Name (FQDN) - Sometimes called Absolute Domain Names.  A domain name that ends in a dot (.), See [RFC 1035](http://tools.ietf.org/html/rfc1035) for details.  A zone name must be a FQDN.
- Relative Domain Name (RelDN) - A domain name that does not end in a dot (.).  This name is concatenated with the zone's name to create a FQDN.  See [RFC 1035](http://tools.ietf.org/html/rfc1035) for details.

## Conventions
### International Domain Names
International domain names are specified using [punycode](http://www.ietf.org/rfc/rfc3492.txt). See [RFC 3492](http://tools.ietf.org/html/rfc3492) for more details.

## RRSet Format

    {
        "zoneName" : "domain.name.",
        "ownerName": "a.domain.name.",
        "class" : "IN",
        "rrtype": "A (1)",
        "ttl": 300,
        "rdata": [
            "1.2.3.4",
            "5.6.7.8",
            "9.10.11.12"
        ],
        "profile": {
            "@context": "http://schemas.ultradns.com/RDPool.jsonschema",
            "order": "FIXED"
        }
    }
 

### Valid values for fields


| Field      | Meaning      | Type  | Valid Values  |
|:-----------|:-------------|:-----:|:--------------|
| zoneName   | The domain name of the apex of the zone. | string (FQDN) | The name of the zone that contains this RRSet.  Not present (and ignored if present) if this RRSet is embedded inside of a [Zone List Format](#Zone List Format) or [Compact Zone Format](#Compact Zone Format). |
| ownerName     | The domain name of the owner of the RRSet | string (FQDN or RelDN) | If a FQDN, must be contained within the zone name FQDN.  The special value "@" can be used to represent an owner name at the apex of the zone.  Not present (and ignored if present) if this RRSet is embedded inside of a [Compact Zone Format](#Compact Zone Format).|
| class | The class of the DNS record | string | Optional.  Valid values are IN, CH, or HS.  If not present, defaults to IN. |
| rrtype | Resource record type for the RRSet. | string | Resource Record Type. Contains the resource record type name (A, AAAA, SRV, etc.).  Optionally, a space, an opening parenthesis, the resource record type value (a number between 1 and 65535), and a closing parenthesis are also included. If there is no defined name for the resource record type value, the string TYPE and the resource record type value replaces the resource name (ex. TYPE1100 (1100)). Not present (and ignored if present) if this RRSet is embedded inside of a [Compact Zone Format](#Compact Zone Format). |
| ttl     | TTL for RRSet    | integer | between [0 and 2147483647](http://tools.ietf.org/html/rfc2181), inclusive. If ttl is not specified, then the TTL for the zone's negative answers is to be used. |
| rdata  | The data for the records in the RRSet | array | List of data fields in presentation format for the specified rrtype.|
| profile | Extension space to describe vendor-specific information about the RRSet | map | Optional.  If this RRSet does not have any vendor-specific functionality, this field is not included. See [RRSet Profile](#RRSet Profile) for more information. |

### JSON Schema

_TODO: convert to jsonschema_

### RRSet Profile
The profile filed allows a vendor to specify custom metadata for an RRSet.  This allows vendors to represent extensions to the standard DNS concepts and types, yet describe them in a vendor-independent format.  [JSON-LD](http://json-ld.org/) must be used to specify the schema for the vendor-specific information.  A profile must include a field called "@context" whose value is the URI for the JSON Schema that describes the profile's format.

#### Example: UltraDNS Resource Distribution Pools
For example, UltraDNS has a pool type called a "Resource Distribution Pool" or RD Pool.  RD Pools allow customers to specify how the A or AAAA resource records in an RRSet are ordered when the RRSet is returned by the authoritative resolver.  There are three options:

- FIXED - Return the records in the same order each time
- RANDOM - Return the records in a random order each time
- ROUND_ROBIN - On each request, rotate which record is returned first

The current DNS wire and presentation formats provide no place for this information to be stored, so a profile is used to represent this information:

    "profile": {
      "@context": "http://schemas.ultradns.com/RDPool.jsonschema",
      "order": "FIXED"
    }

The JSON Schema to represent this is:

	{
		"$schema": "http://json-schema.org/schema#",
		"title": "RD Pool Profile",
		"type":"object",
		"properties":{
			"@context": {
				"description":"The name for this profile schema",
				"type":"string",
				"enum": ["http://schemas.ultradns.com/RDPool.jsonschema"]
			},
			"order": {
				"description":"The valid values describing how to order records in an RD Pool",
				"type":"string",
				"enum": ["FIXED", "RANDOM", "ROUND_ROBIN"]
			}
		},
		"required": ["@context", "order"]
	}


## Zone List Format

The Zone List Format describes all record sets defined for a zone name in a flat list.  It is convenient for representing paginated RRSets or RRSets that are sorted by an attribute other than owner name.

    {
        "@context": "http://schemas.neustar.biz/Zone.jsonschema",
        "zoneName": "domain.name.",
        "rrsets" : [
            {
                "ownerName": "a.domain.name.",
                "rrtype": "A (1)",
                 "ttl": 300,
                "rdata": [
                    "1.2.3.4",
                    "2.4.6.8",
                    "9.8.7.6"
                ],
                "profile": {
                    "@context": "http://schemas.ultradns.com/RDPool.jsonschema",
                    "order": "FIXED"
                }
            },
            {
                "ownerName": "a.domain.name.",
                "rrtype": "TXT (16)",
                "ttl": 300,
                "rdata": [
                    "The quick brown fox jumped over the lazy dog"
                ]
            }            
        ],
        "profile" : {
        }        
    }


### Valid values for fields

| Field | Meaning      | Type  | Valid Values  |
|:------|:-------------|:-----:|:-------------:|
| @context | The JSON schema that describes this structure | string | http://schemas.neustar.biz/Zone.jsonschema Optional; this value is assumed if the @context field is missing. |
| zoneName |    The zone's domain name.     | string | FQDN. |
| rrsets   | The RRSets in the zone. |array | A list of objects in [RRSet Format](#RRSet Format). |
| profile  | Vendor-specific information about the zone.  | object| See [Zone Profile](#Zone Profile) for more information. |


###  JSON Schema
_TODO: convert to jsonschema_


### Zone Profile
The profile section of a [Zone List Format](#Zone List Format) or a [Compact Zone Format](#Compact Zone Format) allows a vendor to specify custom metadata for a zone.  [JSON-LD](http://json-ld.org/) must be used to specify the schema for the vendor-specific information.  
A profile must include a field called "@context" whose value is the URI for the JSON Schema that describes the profile's format.
            
## Compact Zone Format
In order to avoid repetition of the owner name and TTL, another format can be used to describe a zone. The @context field is assigned the value "http://schemas.neustar.biz/CompactZone.jsonschema" in order to differentiate the compact zone format from the zone list format.

As in all JSON formatted data, white space is not significant.

    {
        "@context": "http://schemas.ultradns.com/CompactZone.jsonschema",
        "zoneName": "domain.name.",
        "defaultTTL": 300,
        "ownerNames": {
            "a.domain.name.": {
                "A" : {
                     "rdata": [
                        "1.2.3.4",
                        "2.4.6.8",
                        "9.8.7.6"
                    ],
                    "profile": {
                        "@context": "http://schemas.neustar.biz/RDPool.jsonschema",
                        "order": "FIXED"
                    }
                },
                "TXT": {
                     "rdata": [
                          "The quick brown fox jumped over the lazy dog"
                    ]
                   }
            },
            "b": {
              "A": {
                    "ttl":500,
                    "rdata": [
                        "12.34.56.78"
                    ]
              }
            }
        }
    }
    

### Valid values for fields

| Field      | Meaning       | Type  | Valid Values  |
|:----------:|:-------------:|:-----:|:-------------:|
|@context    | The JSON schema that describes this structure | string |  http://schemas.neustar.biz/CompactZone.jsonschema |
|zoneName    | The zone's domain name.    | string | FQDN |
|defaultTTL  | The TTL used for all RRSets that do not specify their own TTL. |  integer     |  between [0 and 2147483647](http://tools.ietf.org/html/rfc2181), inclusive. |
|ownerNames  |    The owner names defined within the zone.| object | See [The ownerNames Field](#The ownerNames Field) for more information.|
|profile     | Vendor-specific information about the zone | object | See [Zone Profile](#Zone Profile) for more information.|

#### The ownerNames Field
The value for the ownerNames field is a JSON object.  

The field names in the ownerNames object are valid domain names, either FQDNs or RelDNs.  The special field name "@" represents an owner name at the apex of the zone.  It is an error to repeat a FQDN, repeat a relative domain name, repeat "@", to have defined both "@" and a FQDN for the zone, or to have a relative domain name that resolves to a defined FQDN. The ownerNames section can be left empty if no records are defined for the zone.   

The values for the fields in the ownerNames object are also objects.  The field names are resource record type names.  For resource record type values with no defined resource record type names, the string TYPE and the resource record type value replaces the resource name (ex. TYPE1100).

The values are objects in the [RRSet Format](#RRSet Format).

###  JSON Schema
_TODO: convert to jsonschema_


## DNSSEC Extensions
### RRSIG
In a signed zone, there are one or more RRSIG resource records for each RRSet.  Rather than specify the RRSIGs separately from the associated RRSet, an optional RRSIG section can be added to an RRSet in a signed zone:

    {
        "zoneName" : "domain.name.",
        "ownerName": "a.domain.name.",
        "class" : "IN",
        "rrtype": "A (1)",
        "ttl": 300,
        "rdata": [
            "1.2.3.4"
        ],
        "rrsigs": [
                {
                    "algorithm": 5,
                    "expiration": "20140701130030",
                    "inception": "20130701130030",
                    "keyTag": "12345",
                    "signerName": "domain.name.",
                    "signature": "ABC123YOUANDME="
                }
        ],
    }

| Field      | Meaning      | Type  | Valid Values  |
|:-----------|:-------------|:-----:|:--------------|
| rrsigs  | List of resource record signatures.  | array | RRSIG information for the RRSet. Present only for signed zones.|
| rrsigs/algorithm | algorithm used for rrsig | unsigned integer | unsigned, positive 8-bit integer in base 10. 1, 2, 3, 4, 5, 252, 253, and 254 are valid values. See [RFC 4034](http://www.ietf.org/rfc/rfc4034.txt) for valid value definitions. |
| rrsigs/expiration | expiration date/time for rrsig | string |    YYYYMMDDHHmmSS in UTC (see [RFC 4034 3.2](http://www.ietf.org/rfc/rfc4034.txt)) |
| rrsigs/inception | inception date/time for rrsig | string |    YYYYMMDDHHmmSS in UTC (see [RFC 4034 3.2](http://www.ietf.org/rfc/rfc4034.txt)) |
| rrsig/keyTag    | The key tag value of the DNSKEY Resource Record. | unsigned integer | Unsigned 16-bit integer in base 10, which ranges from 0 to 65535. See [RFC 4034 3.2](http://www.ietf.org/rfc/rfc4034.txt) on how to calculate. |
| rrsigs/signerName  | domain name that's signing this owner name.| string | Not present (and ignored if present) if this RRSet is embedded inside of a [Zone List Format](#Zone List Format) or [Compact Zone Format](#Compact Zone Format). |
| rrsigs/signature | The cryptographic signature. | string |  Base-64 encoded signature |  

_Note: Several of the fields in the RRSIG wire and presentation format are not part of this structure.  This is because the type covered, label count, and original ttl are already present in the RRSet structure._

Since an RRSIG is a standard resource record type, it is also legal to define an RRSIG RRSet separately from the associated RRSet.  In this case, the RRSIGs are specified like any other RRSet in either the [Zone List Format](#Zone List Format) or the [Compact Zone Format](#Compact Zone Format).

### NSEC, NSEC3 and DNSKEY
NSEC, NSEC3, and DNSKEY RRSets are specified like other standard RRSets.  No special support is provided in this specification for their representation.

### Private keys for ZSK and KSK
There is nothing in this specification to describe how the private keys for the ZSK and KSK records are transmitted.  A vendor can use the profile field in the zone formats to define a custom system for security transmitting the keys.

### DS
There is nothing in this specification to describe how a DS record is transmitted to the DNS provider for a parent zone or registrar.  

## API Recommendations

### Resource URI
    /v1/zones/{zonesName}/rrsets/{rrtype}/{ownerName}

Path Params

| Param    | Meaning |
|:-------------:|:-----:|
| zoneName    | The domain name. The trailing . is optional, but recommended.| 
| rrtype | The type of the records. This can be expressed either as a 16-bit unsigned positive number or as a standard resource record name (A, AAAA, TXT, SRV, etc). The special rrtype of "ANY" is used in combination with a specified ownerName to return all resource records of all types at a specified owner name. |
| ownerName | The owner name. This name may be fully qualified or partial. A fully qualified name must end in a `.` (trailing period). A partial name must not end in a `.` (trailing period)

### Zone management
This specification does not cover the vendor-specific zone information needed to create a new zone or update a zone's metadata.  However, the standard Resource URI can be used to refer to the per-vendor zone metadata resource.

#### Creating a Zone
POST to /v1/zones

#### Updating a Zone
PUT or PATCH to /v1/zones/{zoneName}

#### Retrieving Zone Metadata
GET from /v1/zones/{zoneName}

##### Deleting a Zone
DELETE from /v1/zones/{zoneName}

### Retrieving Zone Contents
These calls return the zone in [Zone List Format](#Zone List Format).

#### Full Zone Content
GET from /v1/zones/{zoneName}/rrsets

#### Partial Zone Content

##### To get all record sets of a specified type for a zone:
GET from /v1/zones/{zoneName}/rrsets/{rrtype}

##### To get all record sets at a specified owner name:
GET from /v1/zones/{zoneName}/rrsets/ANY/{ownerName}

##### To get the record set at a specified owner name and type:
GET from /v1/zones/{zoneName}/rrsets/{rrtype}/{ownerName}

### RRSet Modifications
These calls use a [RRSet Format](#RRSet Format) as the data type.  The zoneName, ownerName, and rrType fields in the RRSet Format will be ignored if present, as this information is already available via the URI.

#### Create an RRSet
POST to /v1/zones/{zoneName}/rrsets/{rrType}/{ownerName}

#### Update an RRSet
PUT or PATCH to /v1/zones/{zoneName}/rrsets/{rrType}/{ownerName}

#### Delete an RRSet 
DELETE from /v1/zones/{zoneName}/rrsets/{rrType}/{ownerName}

### Batch Operations

#### Updating RRSets in a zone
In order to update all RRSets in a zone, PUT or PATCH to /v1/zones/{zoneName}/rrsets.  Use either the [Zone List Format](#Zone List Format) or [Compact Zone Format](#Compact Zone Format).  PATCH allows updates to existing RRSets or adding new RRSets.  PUT requires all RRSets in the zone to be listed; any RRSets not included are deleted.

Another option is to use the JSON-PATCH proposed standard: [json-patch](http://tools.ietf.org/html/draft-ietf-appsawg-json-patch-10), which allows additions, modifications, and deletes to be expressed within a PATCH.

## Coelacanth
[Coelacanth](http://en.wikipedia.org/wiki/Coelacanth) is a "living fossil".

## Contributors
Google group for [Coelacanth](https://groups.google.com/forum/#!forum/coelacanth-dev)

Neustar

* [Jonathan Bodner](https://github.com/jonbodner)
* [Jeff Damick](https://github.com/jdamick)
* [Edward Lewis](http://blog.neustar.biz/author/elewis/)
* [Vijay Poliboyina](http://www.linkedin.com/pub/vijay-poliboyina/a/a7a/745)


UltraDNS support is not implied or guaranteed by use of this project. We will happily take patches or pull requests and address issues as much as possible.

## Version
0.3 - Match the implementation in the beta of UltraDNS's REST API for RRSet Format, Zone List Format, and API.  Update Compact Zone Format.  Introduce section on DNSSEC support.

0.2 - Incorporate some comments from https://groups.google.com/d/msg/coelacanth-dev/8ivUlQ_N2vk/DvBPpkIaD6QJ

0.1 - Initial release
