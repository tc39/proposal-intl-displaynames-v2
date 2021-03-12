# Proposal: Intl.DisplayNames V2

## Stage 
Stage 2

* Advanced to [Stage 1](https://docs.google.com/presentation/d/11Ch4Y9yYzMJjznX478Y0QbbCGiOAXbOzLjpYnMH9eck/edit#slide=id.g98718c7875_0_0) in TC39 2020-09 meeting.
* Advanced to [Stage 2](https://docs.google.com/presentation/d/1Fsr1BhK11rCNgwEMGtwrMq5arpQv2hlEsA2SRiZSTx4/) in [TC39 2021-01-25~28 meeting](https://github.com/tc39/agendas/blob/master/2021/01.md).
* Plan to propose to Stage 3 in TC39 2021 April meeting



## Champion
* Frank Tang @FrankYFTang

## Reviwers
* TBD

## Editors
* TBD

## Motivation
Main motivation for Intl.DisplayNames project was to enable developers to get translation of language, region or script display names on the client. Translation of languages, regions or script display names requires large amount of data to transmit on the network, which is already available in most browsers. These display name translations also carry steep data size penalty for developers. This API will allow web developers to shrink the size of their HTML and/ or ECMA script code without the need to include the human readble form of display names and therefore reduce the download size to decrease latency. Also, this API will reduce the localization cost for the web developers. Our goal is to expose this data through Intl API for use in e.g. language, region and script pickers, etc.

In Intl.DisplayNames API, we already cover language, region, script, and currency. This proposal enhance the Intl.DisplayNames API and cover more.

Please also see prior work of the first version of Intl.DisplayNames on [the Intl.DisplayNames repo](https://github.com/tc39/proposal-intl-displaynames/)

## Scoped Enhancements
  * [Unit Names](https://github.com/tc39/proposal-intl-displaynames/issues/34)
  * [Calendar Names](https://github.com/tc39/proposal-intl-displaynames/issues/69)
  
During 2021-01 ECMA402 meeting we decided not to include Weekday, Month, TimeZone, and Numbering System. 
During 2021-03-11 ECMA402 meeting we decided to add back dateTimeField. 

## Examples
### Dialect Handling
```
ftang@ftang4:~/v8/v8$ out/x64.release/d8 --harmony_intl_displaynames_v2
V8 version 9.1.0 (candidate)
d8> dn1 = new Intl.DisplayNames("en", {type: "language"})                  
[object Intl.DisplayNames]
d8> dn1.of("en")
"English"
d8> dn1.of("en-GB")
"British English"
d8> dn1.of("en-US")
"American English"
d8> dn1.of("en-AU")
"Australian English"
d8> dn1.of("en-CA")
"Canadian English"
d8> dn1.of("zh")
"Chinese"
d8> dn1.of("zh-Hant")
"Traditional Chinese"
d8> dn1.of("zh-Hans")
"Simplified Chinese"

// Same as above
d8> dn2 = new Intl.DisplayNames("en", {type: "language", dialectHandling: "dialectName"})
[object Intl.DisplayNames]
d8> dn2.of("en")
"English"
d8> dn2.of("en-GB")
"British English"
d8> dn2.of("en-US")
"American English"
d8> dn2.of("en-AU")
"Australian English"
d8> dn2.of("en-CA")
"Canadian English"
d8> dn2.of("zh")
"Chinese"
d8> dn2.of("zh-Hant")
"Traditional Chinese"
d8> dn2.of("zh-Hans")
"Simplified Chinese"

// Now switch to standard name
d8> dn3 = new Intl.DisplayNames("en", {type: "language", dialectHandling: "standardName"})
[object Intl.DisplayNames]
d8> dn3.of("en")
"English"
d8> dn3.of("en-GB")
"English (United Kingdom)"
d8> dn3.of("en-AU")
"English (Australia)"
d8> dn3.of("en-CA")
"English (Canada)"
d8> dn3.of("en-US")
"English (United States)"
d8> dn3.of("zh")
"Chinese"
d8> dn3.of("zh-Hant")
"Chinese (Traditional)"
d8> dn3.of("zh-Hans")
"Chinese (Simplified)"
```

## Discussed Scopes during Stage 1

* Extending Types:
  * ~~[Weekday and Month Names](https://github.com/tc39/proposal-intl-displaynames/issues/75)~~
  * [Unit Names](https://github.com/tc39/proposal-intl-displaynames/issues/34)
  * ~~[TimeZone Names](https://github.com/tc39/proposal-intl-displaynames/issues/17)~~
  * [Calendar Names](https://github.com/tc39/proposal-intl-displaynames/issues/69)
  * ~~[Numbering System Names](https://github.com/tc39/proposal-intl-displaynames/issues/68)~~
  * ~~Quarter~~
  * ~~Day Period~~
  * ~~Date Time Field~~
* Enhance Features:
  * [Supporing Dialect](https://github.com/tc39/proposal-intl-displaynames/issues/20)

## Discussed Scopes during Stage 2
  * 2021-03-11: Add back dateTimeField

## Entrance Criteria For Stage 3
* All Stage 2 Criterias
* Complete spec text
* Designated reviewers have signed off on the current spec text
* All ECMAScript editors have signed off on the current spec text

## Entrance Criteria For Stage 2
* All Stage 1 Criterias
* Initial spec text

## Entrance Criteria For Stage 1

* Identified “champion” who will advance the addition: **DONE- @FrankYFTang**
* Prose outlining the problem or need and the general shape of a solution
* Illustrative examples of usage
* High-level API
* Discussion of key algorithms, abstractions and semantics
* Identification of potential “cross-cutting” concerns and implementation challenges/complexity
* A publicly available repository for the proposal that captures the above requirements: **DONE https://github.com/tc39/intl-displaynames-v2**

## Experimentals
* [V8 Prototype](https://chromium-review.googlesource.com/c/v8/v8/+/2335890)
  * Partial implementation of weekdays , month, unit, timeZone, calendar, numberingSystem.

