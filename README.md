# DNS JSON Spec

The goal of this project is openly define a DNS JSON Specification that will facilitate interoperability between DNS Providers.

[UltraDNS](http://www.neustar.biz/enterprise/dns-services) is currently proposing the following format, but we would like to receive feedback from the community and format that anyone can implement.



## RRSet Structure

    {
    	"name": "default",
    	"version": 1,
    	"zoneName" : "domain.name.",
    	"ownerName": "a.domain.name.",
    	"rrtype": "A",
    	"ttl": 300,
    	"rdata": [
    		"1.2.3.4",
    		"2.4.6.8",
    		"9.8.7.6"
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
    	]
    }
 

### Valid values for fields


| Field      | Meaning      | Type | Valid Values  |
|:-------------:|:-------------:|:-----:|:-----:|	
| name  | A user-defined name for the RRSet	 | string | unique for all RRSets with same owner name and type. For non-pools, this value is always "default". This name must be composed of the characters a-zA-Z0-9_- . If the name field is not specified, "default" is assumed. |
| version	 | version of RRSet. Each update should increase this value.	| unsigned integer | positive 128-bit integer in base 10. Leading zeros must be left off.
| zoneName | The domain name of the apex of the zone. | string | valid domain name. International domain names are specified using [punycode](http://www.ietf.org/rfc/rfc3492.txt). See RFC 3492 for more details. Not present (and ignored if present) if this RRSet is embedded inside of an Owner structure |
| ownerName	 | The domain name of the owner of the RRSet | string |	Valid domain name. International domain names are specified using punycode. See RFC 3492 for more details. Not present (and ignored if present) if this RRSet is embedded inside of an Owner structure |
| rrtype | Resource record type for the RRSet. | string |	Resource Record Type. Either a number between 1 and 65535 or a known resource record name (A, AAAA, SRV, etc.). Not present (and ignored if present) if this RRSet is embedded inside of an Owner structure. |
| ttl	 | TTL for RRSet	| integer | between [0 and 2147483647](http://tools.ietf.org/html/rfc2181), inclusive. If ttl is not specified, the default ttl for the zone will be used. |
| rdata  | The data for the records in the RRSet | array |	List of data fields in presentation format for the specified rrtype. |
| rrsigs  | Present only for signed zones. | array | RRSIG information for the RRSet.	 |
| rrsigs/algorithm | algorithm used for rrsig | unsigned integer |	unsigned, positive 8-bit integer in base 10. 1, 2, 3, 4, 5, 252, 253, and 254 are valid values. See RFC 4034 for valid value definitions. |
| rrsigs/expiration | expiration date/time for rrsig | string |	YYYYMMDDHHmmSS in UTC (see [RFC 4034 3.2](http://www.ietf.org/rfc/rfc4034.txt)) |
| rrsigs/inception | inception date/time for rrsig | string |	YYYYMMDDHHmmSS in UTC (see [RFC 4034 3.2](http://www.ietf.org/rfc/rfc4034.txt)) |
| rrsig/keyTag	| | unsigned integer | Unsigned 16-bit integer in base 10. See [RFC 4034 3.2](http://www.ietf.org/rfc/rfc4034.txt) on how to calculate. |
| rrsigs/signerName  | domain name that's signing	domain name. | string | Optional if this is embedded inside of a Zone structure |
| rrsigs/signature | | string |  Base-64 encoded signature |  |


_TODO: convert to jsonschema_

_Note: that several of the fields in the RRSIG wire and presentation format are not part of this structure.  This is because the type covered, label count, and original ttl are already present in the RRSet structure._



## Owner Structure

The Owner structure describes the record sets defined at a particular domain name.

    {
    	"name": "a.domain.name.",
    	"zoneName": "domain.name.",
    	"rrtypes": {
    		"A": {
    			"rrsets": [
    				{
    					"version": 1,
    					"ttl": 300,
    					"rdata": [
    						"1.2.3.4",
    						"2.4.6.8",
    						"9.8.7.6"
    					]
    				}
    			],
    			"profile": {
    				"@context": "http://neustar.biz/RDPool.jsonschema",
    				"order": "FIXED"
    			}
    		},
    		"16": {
    			"rrsets": [
    				{
    					"version": 1,
    					"ttl": 300,
    					"rdata": [
    						"The quick brown fox jumped over the lazy dog"
    					]
    				}
    			],
    			"profile": {
    			}
    		}
    	},
    	"nsec3s": [
    		{
    			"algorithm": 1,
    			"flags": 1,
    			"iterations": 1,
    			"salt": "DEADBEEF",
    			"nextOwnerName": "ABCDEF"
    		}
    	]
    }


### Valid values for fields

| Field      | Meaning      | Type | Valid Values  |
|:------:|:-------------:|:-----:|:-----:|	
| name | The domain name for the owner	| string | valid domain name.  International domain names are specified using punycode. See [RFC 3492](http://www.ietf.org/rfc/rfc3492.txt) for more details. Not present (and ignored if present) if this Owner structure is embedded inside of a Zone structure |
| zoneName |	The zone's domain name 	| string | valid domain name.  International domain names are specified using punycode. See [RFC 3492](http://www.ietf.org/rfc/rfc3492.txt) for more details. Not present (and ignored if present) if this Owner structure is embedded inside of a Zone structure. |
| rrtypes	| The resource record types defined at the owner name	| object | The keys are valid resource records types, either expressed as numbers or names.  It is an error to repeat a number or name, or to use both a number and a name that refer to the same resource record type (ex. "1" and "A"). |
| rrtypes/{type}/rrsets |	The RRSets defined for the owner name and resource record type. |array|	List of [RRSet structures](RRSet Structure). |
| nsec3s	| Present only for a signed domain. The NSEC3 records defined for the owner name.	A list of NSEC3s. Multiple NSEC3s are allowed for key rollover purposes (the salt, in particular, could be changing) |
| nsec3s/algorithm |	Algorithm used to generate the hash. | integer | See [RFC 5155](http://www.ietf.org/rfc/rfc5155.txt) for more details.	An integer between 0 and 255, inclusive |
| nsec3s/flags |	Flags for NSEC3 See [RFC 5155](http://www.ietf.org/rfc/rfc5155.txt) for more details.	| integer | An integer between 0 and 255, inclusive |
| nsec3s/iterations	| The number of iterations used to generate the hash | integer |	An integer between 0 and 65535, inclusive |
| nsec3s/salt	| The salt used to generate the hash for this owner's name.	| string | Hexidecimal string. If there is no salt for an NSEC3, this field is left out. |
| nsec3s/nextOwnerName | The hash of the next owner name in alphabetical order.| string | base32-encoded string. |
| rrtypes/{type}/profile | Vendor-specific information about how to interpret the RRSets for this owner name and record type	| object | any key-value pairs defined are vendor-specific. If profile is empty, then it is assumed that there is a single RRSet called "default" present for the host name and record type, and that on a request all records are returned in a round-robin order. It is invalid to have an empty profile _AND_ more than one RRSet defined for a host name and record type. See Profile section. |


_TODO: convert to jsonschema_


#### Profile

The profile section of an [Owner Structure](#Owner Structure) allows a vendor to specify custom metadata for an owner name or RRSet within that owner name.  [JSON-LD](http://json-ld.org/) should be used to specify the schema for the vendor-specific information.  For example, to describe the RD (Resource Distribution) pools supported by UltraDNS, the following content would be used in a profile:

			"profile": {
				"@context": "http://schemas.neustar.biz/RDPool.jsonschema",
				"order": "FIXED"
			}
			
## Zone Structure

    {
    	"name": "domain.name.",
    	"ownerNames": {
    		"a.domain.name.": {
    			"rrtypes": {
    				"A": {
					
    				}
    			} 
    		},
    		"b": {
    		}
    	},
    	"profile": {
    	}
    }
    

### Valid values for fields

| Field      | Meaning      | Type | Valid Values  |
|:-------:|:-------------:|:-----:|:-----:|
|name	| The domain name of the zone	A domain name. | string | International domain names are specified using punycode. See RFC 3492 for more details.
|ownerNames |	The owner names defined within the zone.	| object | The keys are valid domain names. Both fully qualified domain names and relative domain names are allowed. Fully qualified domain names must end in a `.` (trailing period). Relative domain names must NOT end in a `.` (trailing period). It is an error to repeat a FQDN, repeat a relative domain name, or to have a relative domain name that resolves to a defined FQDN. The ownerNames section can be left empty if no records are defined for the zone. |
profile | Vendor-specific information about the zone | object | Any key-value pairs defined are vendor-specific. See Profile section|

#### Profile

The profile section of the [Zone structure](#Zone Structure) is required, but can be empty.  An empty profile section means that no vendor custom metadata is defined for this zone.


## API Recommendations

### Resource URI

    /v1/{className}/zones/{zonesName}/records/{rrtype}/{ownerName}/{rrSetId}

Path Params

| Param	| Meaning |
|:-------------:|:-----:|
| className |	IN, CH, HS, CS (Obsolete). Most commonly, IN is used. |
| zoneName	| The domain name. The trailing . is optional, but recommended.| 
| rrtype | The type of the records. This can be expressed either as a 16-bit unsigned positive number or as a standard resource record name (A, AAAA, TXT, SRV, etc). The special rrtype of "ANY" is used in combination with a specified ownerName to return all resource records of all types at a specified owner name. |
| ownerName | The owner name. This name may be fully qualified or partial. A fully qualified name must end in a `.` (trailing period). A partial name must not end in a `.` (trailing period)
| rrSetName	| The name of the RRSet. This name is unique for the record type and owner name. If there are no dynamic features for the owner name and record type, the rrSetId should be "default".|

### Creating a Zone

### Retrieving Zone Contents

#### Full Zone Content

#### Partial Zone Content

### Updating a Zone

#### Updating a complete zone

In order to update a complete zone, specify all data in a zone from the [Zone Structure](#Zone Structure) object down.  Deletes of an owner or an RRSet are performed by not including them in the structure.

#### Updating a single RRSet or multiple RRSets at a single owner name

In order to update a single RRSet, use an [RRSet Structure](#RRSet Structure) object only.  

If you are specifying vendor-specific data and/or multiple RRSets for for a single owner name, use an [Owner Structure](#Owner Structure). Deletes of RRSets are performed by not including them in the structure.


#### Updating multiple RRSets
In order to modify multiple RRSets within a single operation, the recommendation is to use the JSON-PATCH proposed standard: [json-patch](http://tools.ietf.org/html/draft-ietf-appsawg-json-patch-10).




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

