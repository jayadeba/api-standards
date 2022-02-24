# Common Types


Please refer to [documentation](../../../api-style-guide.md#common-types) to find more details about Common Types. For more details on the address type refer to [README](README_address.md) dedicated to the type.


#### Extend Common Types (if needed)

API developers are encouraged to use Common Types as-is. However, in cases where you need to extend a type, there are multiple ways to achieve this as described below. Note that each of the following approach comes with its own trade-off.

1. Use composition. That is, `$ref` a Common Type in one of the properties of your type and add more properties you need. 
2. Use [`allOf`](http://json-schema.org/latest/json-schema-validation.html#anchor82) keyword as described in [Structuring a complex schema](http://spacetelescope.github.io/understanding-json-schema/structuring.html#extending) using [JSON Schema Draft 4][1]. Keyword `allOf` in draft #4 replaces the keyword `extends` of draft #3. Note that not all tools recognize/treat the `allOf` keyword correctly at the time of writing this README (Dec 2014).
3. Make a copy of Common Type and then add your own properties. This goes against the whole notion of `reuse`, however, it would work and tools would not have any problems with it. 
 

[1]: http://json-schema.org/latest/json-schema-core.html "JSON Schema: core definitions and terminology"
[7]: http://tools.ietf.org/html/draft-zyp-json-schema-03 "A JSON Media Type for Describing the Structure and Meaning of JSON Documents"
[11]: schema/json/draft-04/src/main/json/address.json "address.json"
[12]: schema/json/draft-04/src/main/json/address_global.json "address_global.json"
[13]: schema/json/draft-04/src/main/json/address_portable.json "address_portable.json"
[14]: https://github.com/googlei18n/libaddressinput/wiki/AddressValidationMetadata "i18napis"
[15]: https://www.w3.org/TR/html51/sec-forms.html#autofill-field "HTML 5.1 autofill"
[16]: https://www.informatica.com/content/dam/informatica-com/global/amer/us/collateral/other/addressdoctor-cloud-2_user-guide.pdf "Address Doctor"
[17]: https://developers.google.com/maps/documentation/geocoding/intro#Types "Google Maps Geocoding API"
[18]: http://microformats.org/wiki/adr "hcard"

