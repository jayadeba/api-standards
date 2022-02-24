---
layout: master
title: Postal Address
---

* [General Design](#general-design)
* [Why admin\_area instead of state, province, city, locality?](#why-admin_area)
* [address\_portable\.json fields and their use](#address_portablejson-fields-and-their-use)
* [Example addresses](#example-addresses)
* [Mapping of portable fields to other standards](#mapping-portable-fields)
* [Address validation](#address-validation)
* [Detail needs for specific scenarios \- address\_details](#detail-needs-for-specific-scenarios---address_details)
* [Mapping of detail fields](#mapping-of-detail-fields)


<h1 id="portable-address">Portable Postal Address Schema</h1>

    
APIs extensively use address. This document describes a portable address schema. 

This schema is designed to be

* backward compatible with [address.json](https://developer.paypal.com/docs/api/payments/#definition-address) schema and [hcard][1] address microformats
* forward compatible with Google opensource [address validation metadata][2] and W3 [HTML5.1 autofill fields][3].
* allows mapping to and from the Trinity Address Normalization Service (ANS) or [AddressDoctor][4]


## General Design

The general design intends to keep a fairly generic nomenclature so as not to confuse with the many names and synonyms for the same entities. For example, just `admin_area_level_1` is used instead of state or province or county, or prefecture.

In addition to `country_code` and `postal_code` there are only three parts:

* **street address**, composed of `address_line_1` through `address_line_3`. This would be filled in with items such as the street address, apartment number, building information, or additional guidance.
* **administrative area**, composed of `admin_area_1` through `admin_area_4`. This would be composed of province, city, suburb information as needed for each country. Note that these are presented in reverse order in this schema, as in most Western address formats, administrative areas are presented from smallest (`admin_area_4`) to largest (`admin_area_1`), and follows the order used in HTML5.1.
* **address details**, needed for specific applications that need non-portable, but detailed normalized information for risk, compliance or other scenarios. This is explained in more detail in a separate section below.

<h2 id="why-admin_area">Why admin_area instead of state, province, city, locality?</h2>

In the design of this schema, there was much discussion that we might sacrifice developer usability by not using terms such as 'city' or 'locality', and 'state' or 'province'.  The regional descriptors 'city'/'locality' and 'state'/'province' are primarily for Western conventions and do not capture the multi-level regional segmentation needed for Brazil, China, India, Indonesia, and many other places.  Therefore, the decision was made to mirror the generic terms used in [HTML5.1 autofill fields][3] which breaks regional areas down from `1` (largest) to `4` (smallest).  These are further described in sections below.

## [address_portable.json][7] fields and their use

* **address_line_1**: Line 1 of the Address (eg. number, street, etc). Example: '173 Drury Lane'. Usually required for compliance and risk checks. This should contain the full address line 1.
* **address_line_2**: Optional line 2 of the Address (eg. suite, apt #, etc.)
* **address_line_3**: Optional Line 3 of the Address if needed. Examples include Street Complement for Brazil, direction text such as 'next to Walmart' or landmark in Indian address.
* **admin_area_4**: May be neighborhood, ward or district. Smaller than admin_area_level_3. Smaller than sub_locality. May be Postal Sorting Code used in Guernsey, and many French territories such as French Guiana, or fine-grained administrative levels in China.
* **admin_area_3**: Usually sub-locality, suburb, neighborhood, or district. Smaller than admin_area_level_2. This could be suburb, or Bairro/Neighborhood in Brazil. In India, street name information is often not available, but a sublocality/district can be a very small area and used in `admin_area_3`.
* **admin_area_2**: Usually city, town or village. Smaller than admin_area_level_1.
* **admin_area_1**: Usually corresponds to Province, State, or ISO-3166-2 subdivision. Highest level sub-division of a country. This will be county in UK, state in USA, province in Canada, prefecture in Japan and kanton in Switzerland. This is expected to be in the form that would appear in address formatted for postal delivery, e.g. 'CA' and not 'California'
* **postal_code**: Postal code, ZIP code, CEP, etc.
* **country_code**: the two-letter ISO territory code as specified by `country_code.json`.
* **address_details object**: Non-portable additional address details sometimes needed for compliance, risk, and other scenarios where fine-grain address information may be needed. Not portable with common 3rd party and opensource. Redundant with core fields. E.g., `address_line_1` is usually a combination of `address_details.street_number` + `address_details.street_name` + `address_details.street_type`
    * **street_number**: Street number, e.g. '173' from '173 Drury Lane.
    * **street_name**: Street name.  Just 'Drury' from '173 Drury Lane'. If street type is not available e.g. from AddressDoctor, then this should contain both the street name and type as in 'Drury Lane'.
    * **street_type**: The street type of the address. 'Street', 'Lane', 'Boulevard', 'Court', etc. Note that AddressDoctor may not easily resolve street type.
    * **delivery_service**: Delivery service. Post Office Box, Bag number, Post Office Name.
    * **building_name**: Premise. The building name or number. Example: 'Craven House'. Named location, usually a building or collection of buildings with a common name.
    * **sub_building**: Sub-premise. First-order entity below a named building or location, usually a singular building within a collection of buildings with a common name, or could be flat, storey, floor, room and/or apartment.

## Example addresses

A simple [United States address][11] such as 

```
2211 North First Street
Building 17, 17.2.237
San Jose, CA 95131
```
would be represented as

```JSON
{
    "address_line_1": "2211 North First Street",
    "address_line_2": "Building 17, 17.2.237",
    "admin_area_2": "San Jose",
    "admin_area_1": "CA",
    "postal_code": "95131",
    "country_code": "US"
}
```
and from [Brazil][12], where you need a _neighborhood_ within a city (in this example, _Bela Vista_ ):

```
Av. Paulista, 1098, 1º andar, apto. 101
Bela Vista
São Paulo - SP
Brasil
01310-000
```

would be represented as

```JSON
{
    "address_line_1": "Av. Paulista, 1098",
    "address_line_2": "1º andar, apto. 101",
    "admin_area_3": "Bela Vista",
    "admin_area_2": "São Paulo",
    "admin_area_1": "SP",
    "postal_code": "01310-000",
    "country_code": "BR"
}
```

<h2 id="mapping-portable-fields">Mapping of portable fields</h2>


| [address_portable.json][7] | [HTML 5.1][3] | Google [address metdata][2] | [AddressDoctor][4] |
|:---------------:|:--------------:|:-----------:|:--------------------:|
| address_line_1 | address-line1 | A\[0\] | AddressLine.DeliveryAddressLine[0] |
| address_line_2 | address-line2 | A\[1\] | AddressLine.DeliveryAddressLine\[1\] 
| address_line_3 | address-line3 | A\[2\] | AddressLine.DeliveryAddressLine[2..5] | 
| admin_area_4 | address-level4 | X | AddressLine.CountrySpecificLocalityLine[3..5] or Locality[2..5] |
| admin_area_3 | address-level3 | D | AddressLine.CountrySpecificLocalityLine\[2\] or Locality\[1\] | 
| admin_area_2 | address-level2 | C | AddressLine.CountrySpecificLocalityLine\[1\] or Locality[0], possibly Province\[1\] depending on country |
| admin_area_1 | address-level1 | S | AddressLine.CountrySpecificLocalityLine[0] or Province[0] | state | STATE |
| postal_code | postal-code | Z | AddressElement.PostalCode | 
| country_code | country | | AddressElement.Country |


<h2 id="address-validation">Address validation</h2>

Applications that need high confidence in their address data will want to validate the address objects.  Note that there is a difference between _validation_ and _verification_.  **Validation** makes sure that the address has data in all required fields, that entries for any fields covered by enums for known lists of areas such as provice or state names are valid, and that postal codes pass a regular expression match.  **Verification** makes sure the address matches an actual physical location and is usually accomplished by sending the address object to a normalization or verification service that considers data from local postal authorities.

The structure of [address_portable.json][7] is designed to enable easy validation of address data using the open source Google [address validation metadata][2].

For required fields, the address validation metadata specifies the following codes:
* A - Street address array, at least one line must have data
* S - State/province -  usually admin_area_1 in `address_portable` and html5.1 autocomplete fields
* C - City/Locality -  usually admin_area_2 in `address_portable` and html5.1 autocomplete fields
* D - District/suburb/neighborhood -  usually admin_area_3 in `address_portable` and html5.1
* X - Sort code - usually admin_area_4 in `address_portable`
* Z - ZIP/Postal code

For example, the Google [address validation metadata for the U.S.][9] has the entry
```JSON
"require": "ACSZ"
```

which tells us that at least `A` (`address_line_1`) plus `C` (`admin_area_2`), `S` (`admin_area_1`), and `Z` (`postal_code`) are all required fields. For Singapore, the required fields entry is only
```JSON
"require": "AZ"
```
which tells us that only `address_line_1` and `postal_code` are required.

For postal codes, a regular expression is used to be sure the pattern is well formed.  For the U.S., the pattern in the metadata is 
```JSON
"zip": "(\\d{5})(?:[ \\-](\\d{4}))?"
```
which means the postal code may be five digits, or optionally nine digits with a space or hyphen after the first five digits as in '95131' or '95131-1234'.

Note that if the `"require"` or `"zip"` field are missing from a country record, then it should inherit from the [ZZ default metadata][10] which has a validation field of `"require": "AC"`.  This is also the default for PayPal.

PayPal has a few minor variations from the open source metadata, but the PayPal teams are contributing regularly to the open source community to be sure all metadata agrees over time.  Any address validated using the Google open source address validation metadata is guaranteed to validate correctly when passed through PayPal APIs using the `address_portable.json` schema.

<h2 id="detail-needs-for-specific-scenarios---address_details">Detail needs for specific scenarios - address_details</h2>

Some applications need additional detail such as for risk and compliance, and need very specific address details for some countries. This additional detail is not backward compatible with legacy applications and is not portable to open source or HTML 5.1 or easily grabbed from contact databases. But it can be derived from address services such as AddressDoctor or an address normalization service.

The two most common needs are for detailed street address or building information. For this, the `address_details` has been added as a separate object, and included in `address_portable.json`.

It is expected that all information in the `address_details` fields are also duplicated and correctly entered in the `address_portable` fields that are included by `$ref`.
To emphasize the point that this is less-common and non-portable data, it is included as an object instead of in-line with the `address_portable.json` elements.

As an example, the San Jose address from above may be, for very specific, non-portable purposes:

```JSON
{
    "address_line_1": "2211 North First Street",
    "address_line_2": "Building 17, 17.2.237",
    "admin_area_2": "San Jose",
    "admin_area_1": "CA",
    "postal_code": "95131",
    "country_code": "US",
    "address_details": {
        "street_number": "2211",
        "street_name": "North First",
        "street_type": "Street"
    }
}
```

and the Brazil detailed address may become:

```JSON
{
    "address_line_1": "Av. Paulista, 1098",
    "address_line_2": "1º andar, apto. 101",
    "admin_area_3": "Bela Vista",
    "admin_area_2": "São Paulo",
    "admin_area_1": "SP",
    "postal_code": "01310-000",
    "country_code": "BR",
    "address_details": {
        "street_number": "1098",
        "street_name": "Paulista",
        "street_type": "Avenida",
        "building_name": "",
        "sub_building": "1º andar, apto. 101"
    }
}
```

Note again that the `address_details` object should be used only for parsed or user-entered detailed address parts that are needed for very specific applications that can not use the more general address structure.


[1]: http://microformats.org/wiki/adr "hcard"
[2]: https://github.com/googlei18n/libaddressinput/wiki/AddressValidationMetadata "Google open-source address validation metadata"
[3]: https://www.w3.org/TR/html51/sec-forms.html#autofill-field "HTML 5.1 autofill"
[4]: https://www.informatica.com/content/dam/informatica-com/global/amer/us/collateral/other/addressdoctor-cloud-2_user-guide.pdf "Address Doctor"
[7]: draft-04/address_portable.json "address_portable.json"
[8]: https://developers.google.com/maps/documentation/geocoding/intro#Types "Google Maps Geocoding API"
[9]: https://www.gstatic.com/chrome/autofill/libaddressinput/chromium-i18n/ssl-address/data/US "Address validation metadata for the US"
[10]: https://www.gstatic.com/chrome/autofill/libaddressinput/chromium-i18n/ssl-address/data/ZZ "ZZ Default address validation metadata"
[11]: https://pe.usps.com/text/pub28/welcome.htm "United States Postal Addressing Standards"
[12]: http://www.upu.int/fileadmin/documentsFiles/activities/addressingUnit/braEn.pdf "Brazil UPU address format"
