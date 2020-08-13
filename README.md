# Proposal: Intl.DisplayNames V2

## Stage 
Stage 0

## Champion
* Frank Tang @FrankYFTang

## Motivation
Main motivation for Intl.DisplayNames project was to enable developers to get translation of language, region or script display names on the client. Translation of languages, regions or script display names requires large amount of data to transmit on the network, which is already available in most browsers. These display name translations also carry steep data size penalty for developers. This API will allow web developers to shrink the size of their HTML and/ or ECMA script code without the need to include the human readble form of display names and therefore reduce the download size to decrease latency. Also, this API will reduce the localization cost for the web developers. Our goal is to expose this data through Intl API for use in e.g. language, region and script pickers, etc.

In Intl.DisplayNames API, we already cover language, region, script, and currency. This proposal enhance the Intl.DisplayNames API and cover more.

## Possible Enhancements

* Extending Types:
  * [Weekday and Month Names](https://github.com/tc39/proposal-intl-displaynames/issues/75)
  * [Unit Names](https://github.com/tc39/proposal-intl-displaynames/issues/34)
  * [TimeZone Names](https://github.com/tc39/proposal-intl-displaynames/issues/17)
  * [Calendar Names](https://github.com/tc39/proposal-intl-displaynames/issues/69)
  * [Numbering System Names](https://github.com/tc39/proposal-intl-displaynames/issues/68)
  * Quarter
  * Day Period
  * Date Time Field
* Enhance Features:
  * [Supporing Dialect](https://github.com/tc39/proposal-intl-displaynames/issues/20)

## Motivation and Use Cases
### TimeZone Name
Since we already have Intl.DateTimeFormat object in ECMA402, it is "possible" for the developer to get the localized display name of the desired time zone by calling the Intl.DateTimeFormat API. However, it is tricky and error prone.
First of all, the Intl.DateTimeFormat API does not allow us to output only timeZoneName. For example, the following code will use the current time for the output since we do not pass in a Date object:
```
(new Intl.DateTimeFormat("en", {timeZoneName: "long"})).format()
"8/13/2020, Pacific Daylight Time"
```
Notice the same code would output the time zone name differently if it is running in the winter 
```
(new Intl.DateTimeFormat("en", {timeZoneName: "long"})).format(new Date("2020-01-01"))
"12/31/2019, Pacific Standard Time"
```
This implies, if the caller needs to get the localized name for "Pacific Standard Time", it needs to construct a Date object of a known date which is known not in daylight saving time.  
However, this requires the caller to know which particular date is in the range of standard time of a particular time zone. For example, for regions observing daylight saving time in the southern hemisphere, a date during January could fall under the range of daylight saving time while a date during July would fall under standard time. For example:

```
let dtfLA = new Intl.DateTimeFormat("en", {timeZone: "America/Los_Angeles", timeZoneName: "long"})
let dtfSydney = new Intl.DateTimeFormat("en", {timeZone: "Australia/Sydney", timeZoneName: "long"})
let dayInJan = new Date("2019-01-01");
dtfLA.format(dayInJan)
"12/31/2018, Pacific Standard Time"
dtfSydney.format(dayInJan)
"1/1/2019, Australian Eastern Daylight Time"
let dayInJuly = new Date("2019-07-01");
dtfLA.format(dayInJuly)
"6/30/2019, Pacific Daylight Time"
dtfSydney.format(dayInJuly)
"7/1/2019, Australian Eastern Standard Time"
```
As we can see from the code snippet above, we cannot use a fixed single Date object to pass in the format method if we desire to get the name or the standard time or the name for the daylight saving time. It requires us to find a date value for a particular timezone.

This above code example also shows us a second problem- there are no options in  Intl.DateFormat that allow the caller to control to only output the timezone name without other fields. The caller could, of course, call formatToParts instead and then take out the value which type is “timeZoneName”, for example:
```
dtfLA.formatToParts(dayInJan).filter(p => p.type == "timeZoneName").map(p => p.value).join()
"Pacific Standard Time"
dtfSydney.formatToParts(dayInJan).filter(p => p.type == "timeZoneName").map(p => p.value).join()
"Australian Eastern Daylight Time"
dtfLA.formatToParts(dayInJuly).filter(p => p.type == "timeZoneName").map(p => p.value).join()
"Pacific Daylight Time"
dtfSydney.formatToParts(dayInJuly).filter(p => p.type == "timeZoneName").map(p => p.value).join()
"Australian Eastern Standard Time"
```
If we want to put everything together, then we get

```
(new Intl.DateTimeFormat("en", {timeZone: "America/Los_Angeles",timeZoneName: "long"}))
   .formatToParts(new Date("2019-01-01")).filter(p => p.type == "timeZoneName")
   .map(p => p.value).join()
"Pacific Standard Time"  
(new Intl.DateTimeFormat("en", {timeZone: "Australia/Sydney",timeZoneName: "long"}))
   .formatToParts(new Date("2019-01-01")).filter(p => p.type == "timeZoneName")
   .map(p => p.value).join()
"Australian Eastern Daylight Time"
(new Intl.DateTimeFormat("en", {timeZone: "America/Los_Angeles",timeZoneName: "long"}))
   .formatToParts(new Date("2019-07-01")).filter(p => p.type == "timeZoneName")
   .map(p => p.value).join()
"Pacific Daylight Time"
(new Intl.DateTimeFormat("en", {timeZone: "Australia/Sydney",timeZoneName: "long"}))
   .formatToParts(new Date("2019-07-01")).filter(p => p.type == "timeZoneName")
   .map(p => p.value).join()
"Australian Eastern Standard Time"
```
In the other hand, we believe to get the timezone name, we should free developer from 
knowing detail date range of daylight observation of each timezone by using 
Intl.DisplayNames API, as:
```
let zoneNames = new Intl.DisplayNames("en", 
    {type: "timeZoneName", style: "long"})
zoneNames.of("America/Los_Angeles")
"Pacific Standard Time"
zoneNames.of("Australia/Sydney")
"Australian Eastern Standard Time"
```
or
```
let zoneNames = new Intl.DisplayNames("en", 
    {type: "timeZoneName", style: "long", daylightSaving: "false"})
zoneNames.of("America/Los_Angeles")
"Pacific Standard Time"
zoneNames.of("Australia/Sydney")
"Australian Eastern Standard Time"
```
If the caller want to get names of the daylight saving time:
```
let zoneNames = new Intl.DisplayNames("en", 
    {type: "timeZoneName", style: "long", daylightSaving: "true"})
zoneNames.of("America/Los_Angeles")
"Pacific Daylight Time"
zoneNames.of("Australia/Sydney")
"Australian Eastern Daylight Time"
```
If the caller want to get names based on the current status (under July 2020) of each time zone:
```
let zoneNames = new Intl.DisplayNames("en", 
    {type: "timeZoneName", style: "long", daylightSaving: "current"})
zoneNames.of("America/Los_Angeles")
"Pacific Daylight Time"
zoneNames.of("Australia/Sydney")
"Australian Eastern Standard Time"
```
To get the standard time zone names in Traditional Chinese used in Taiwan:
```
let zoneNames = new Intl.DisplayNames("zh-Hant-TW", 
    {type: "timeZoneName", style: "long"})
zoneNames.of("America/Los_Angeles")
"太平洋標準時間"
zoneNames.of("Australia/Sydney")
"澳洲東部標準時間"
```


## Entrance Criteria For Stage 1

* Identified “champion” who will advance the addition: **DONE- @FrankYFTang**
* Prose outlining the problem or need and the general shape of a solution
* Illustrative examples of usage
* High-level API
* Discussion of key algorithms, abstractions and semantics
* Identification of potential “cross-cutting” concerns and implementation challenges/complexity
* A publicly available repository for the proposal that captures the above requirements: **DONE https://github.com/FrankYFTang/intl-displaynames-v2**

## Experimentals
* [V8 Prototype](https://chromium-review.googlesource.com/c/v8/v8/+/2335890)
  * Partial implementation of weekdays , month, unit, timeZone, calendar, numberingSystem.

# TO BE DELETED- FROM TEMPLATE

## Before creating a proposal

Please ensure the following:
  1. You have read the [process document](https://tc39.github.io/process-document/)
  1. You have reviewed the [existing proposals](https://github.com/tc39/proposals/)
  1. You are aware that your proposal requires being a member of TC39, or locating a TC39 delegate to "champion" your proposal

## Create your proposal repo

Follow these steps:
  1.  Click the green ["use this template"](https://github.com/tc39/template-for-proposals/generate) button in the repo header. (Note: Do not fork this repo in GitHub's web interface, as that will later prevent transfer into the TC39 organization)
  1.  Go to your repo settings “Options” page, under “GitHub Pages”, and set the source to the **main branch** under the root (and click Save, if it does not autosave this setting)
      1. check "Enforce HTTPS"
      1. On "Options", under "Features", Ensure "Issues" is checked, and disable "Wiki", and "Projects" (unless you intend to use Projects)
      1. Under "Merge button", check "automatically delete head branches"
<!--
  1.  Avoid merge conflicts with build process output files by running:
      ```sh
      git config --local --add merge.output.driver true
      git config --local --add merge.output.driver true
      ```
  1.  Add a post-rewrite git hook to auto-rebuild the output on every commit:
      ```sh
      cp hooks/post-rewrite .git/hooks/post-rewrite
      chmod +x .git/hooks/post-rewrite
      ```
-->
  1.  ["How to write a good explainer"][explainer] explains how to make a good first impression.

      > Each TC39 proposal should have a `README.md` file which explains the purpose
      > of the proposal and its shape at a high level.
      >
      > ...
      >
      > The rest of this page can be used as a template ...

      Your explainer can point readers to the `index.html` generated from `spec.emu`
      via markdown like

      ```markdown
      You can browse the [ecmarkup output](https://ACCOUNT.github.io/PROJECT/)
      or browse the [source](https://github.com/ACCOUNT/PROJECT/blob/HEAD/spec.emu).
      ```

      where *ACCOUNT* and *PROJECT* are the first two path elements in your project's Github URL.
      For example, for github.com/**tc39**/**template-for-proposals**, *ACCOUNT* is "tc39"
      and *PROJECT* is "template-for-proposals".


## Maintain your proposal repo

  1. Make your changes to `spec.emu` (ecmarkup uses HTML syntax, but is not HTML, so I strongly suggest not naming it ".html")
  1. Any commit that makes meaningful changes to the spec, should run `npm run build` and commit the resulting output.
  1. Whenever you update `ecmarkup`, run `npm run build` and commit any changes that come from that dependency.

  [explainer]: https://github.com/tc39/how-we-work/blob/HEAD/explainer.md
