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

## Shortcoming Without the Improvement in the Status Quo
### Month Names
Since we already have Intl.DateTimeFormat object in ECMA402, it is "possible" for the developer to get the localized display name of the desired month by calling the format method of Intl.DateTimeFormat API with a Date object in that month. However, it is not straightforward under this approach as many may imagine. Consider the following code:
```
let dtf = new Intl.DateTimeFormat("en", {month: "long"})
let monthNames = [
dtf.format(new Date("2020-01-01")),
dtf.format(new Date("2020-02-01")),
dtf.format(new Date("2020-03-01")),
dtf.format(new Date("2020-04-01")),
dtf.format(new Date("2020-05-01")),
dtf.format(new Date("2020-06-01")),
dtf.format(new Date("2020-07-01")),
dtf.format(new Date("2020-08-01")),
dtf.format(new Date("2020-09-01")),
dtf.format(new Date("2020-10-01")),
dtf.format(new Date("2020-11-01")),
dtf.format(new Date("2020-12-01"))]
```
By glancing at the code, it seems monthNames will be an array of the names from January to December, on a based 0 index. However, that is not the case if the client code is running on a timezone with a positive GMT offset:
```
monthNames
["December", "January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November"]
```
This is because the code constructs the Date objects using date in string, and the Intl.DateTimeFormat, constructed without a timeZoneName value in option,  is using the local timezone. So for Intl.DateTimeFormat running on a positive GMT offset timezone, a Date object created with  "2020-01-01" is set to the time of 2020-01-01 00:00:00.000 in UTC, and therefore a time in their local 2019-12-31. So it generates such results that we do not expect. But the same code will generate the correct result if it is running in UTC or any negative GMT offset timezone. There are two hacks to fix that, one is not to use the first day of the month to create the date, instead, use other date, such as:
```
let monthNames = [
dtf.format(new Date("2020-01-02")),
dtf.format(new Date("2020-02-02")),
dtf.format(new Date("2020-03-02")),
dtf.format(new Date("2020-04-02")),
dtf.format(new Date("2020-05-02")),
dtf.format(new Date("2020-06-02")),
dtf.format(new Date("2020-07-02")),
dtf.format(new Date("2020-08-02")),
dtf.format(new Date("2020-09-02")),
dtf.format(new Date("2020-10-02")),
dtf.format(new Date("2020-11-02")),
dtf.format(new Date("2020-12-02"))]
monthNames
["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"]
```
But this look very stange and it would be very complicate to comment the code for such reason. 

Another way to address this is to set the timeZone to "UTC" when construct the Intl.DateTimeFormat object, such as:
```
let dtf = new Intl.DateTimeFormat("en", {month: "long", timeZone: "UTC"})
let monthNames = [
dtf.format(new Date("2020-01-01")),
dtf.format(new Date("2020-02-01")),
dtf.format(new Date("2020-03-01")),
dtf.format(new Date("2020-04-01")),
dtf.format(new Date("2020-05-01")),
dtf.format(new Date("2020-06-01")),
dtf.format(new Date("2020-07-01")),
dtf.format(new Date("2020-08-01")),
dtf.format(new Date("2020-09-01")),
dtf.format(new Date("2020-10-01")),
dtf.format(new Date("2020-11-01")),
dtf.format(new Date("2020-12-01"))]
monthNames
["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"]
```

We believe we should provide month name in Intl.DisplayNames to make it more straightforward, as 
```
let dn = new Intl.DisplayNames("en", {type: "month", style: "long"})
let monthNames = [ dn.of(1), dn.of(2), dn.of(3), dn.of(4), dn.of(5), 
dn.of(6), dn.of(7), dn.of(8), dn.of(9), dn.of(10), dn.of(11), dn.of(12)]
monthNames
["January", "February", "March", "April", "May", "June", "July", "August", "September", "October", "November", "December"]
```
and to get the month names as in Traditional Chinese used in Taiwan:
```
let dn = new Intl.DisplayNames("zh-Hant-TW", {type: "month", style: "long"})
let monthNames = [ dn.of(1), dn.of(2), dn.of(3), dn.of(4), dn.of(5), 
dn.of(6), dn.of(7), dn.of(8), dn.of(9), dn.of(10), dn.of(11), dn.of(12)]
monthNames
 ["1月", "2月", "3月", "4月", "5月", "6月", "7月", "8月", "9月", "10月", "11月", "12月"]
```
The using a Date object with Intl.DateTimeFormat hack is even worse when we want to get the display names for a different calendar system. For example, if we like to get the array of month names in English for the “coptic” calendar, the following code would get us incorrect result:
```
let dtf = new Intl.DateTimeFormat("en", {month: "long", timeZone: "UTC", calendar: "coptic"})
let monthNames = [
dtf.format(new Date("2020-01-01")),
dtf.format(new Date("2020-02-01")),
dtf.format(new Date("2020-03-01")),
dtf.format(new Date("2020-04-01")),
dtf.format(new Date("2020-05-01")),
dtf.format(new Date("2020-06-01")),
dtf.format(new Date("2020-07-01")),
dtf.format(new Date("2020-08-01")),
dtf.format(new Date("2020-09-01")),
dtf.format(new Date("2020-10-01")),
dtf.format(new Date("2020-11-01")),
dtf.format(new Date("2020-12-01"))]
monthNames
["Kiahk", "Toba", "Amshir", "Baramhat", "Baramouda", "Bashans", "Paona", "Epep", "Mesra", "Tout", "Baba", "Hator"]
```
But "Kiahk" is the English name of the fourth month in the [“coptic” calendar system](https://en.wikipedia.org/wiki/Coptic_calendar) and the name of the first month should be “Tout". The problem is the Date object we construct is under gregorian calendar and the Intl.DateTimeFormat will convert that date to the coptic calendar inside the format function, and the conversion result the date “2020-01-01” in a date in the fourth month under the “coptic” calendar system and therefore the Intl.DateTimeFormat output the month name as the fourth month. In order to get the list of the month name in the right order, the developer need to know 12 dates, each correctly reverse convert from the desired month in the “coptic” calendar system back to the Gregorian date and put that into the code. 
```
let dtf = new Intl.DateTimeFormat("en", {month: "long", timeZone: "UTC", calendar: "coptic"})
let monthNames = [
dtf.format(new Date("2019-09-12")),
dtf.format(new Date("2019-10-12")),
dtf.format(new Date("2019-11-11")),
dtf.format(new Date("2019-12-11")),
dtf.format(new Date("2020-01-10")),
dtf.format(new Date("2020-02-09")),
dtf.format(new Date("2020-03-10")),
dtf.format(new Date("2020-04-09")),
dtf.format(new Date("2020-05-09")),
dtf.format(new Date("2020-06-08")),
dtf.format(new Date("2020-07-08")),
dtf.format(new Date("2020-08-07"))]
monthNames
["Tout", "Baba", "Hator", "Kiahk", "Toba", "Amshir", "Baramhat", "Baramouda", "Bashans", "Paona", "Epep", "Mesra"]
```
As you can see, this makes the code using such hack pretty complicated and hard to understand. By using Intl.DisplayNames, the code would looks straightforward as:
```
let dn = new Intl.DisplayNames("en", {type: "month", style: "long", calendar: "coptic"})
let monthNames = [ dn.of(1), dn.of(2), dn.of(3), dn.of(4), dn.of(5), 
dn.of(6), dn.of(7), dn.of(8), dn.of(9), dn.of(10), dn.of(11), dn.of(12)]
monthNames
["Tout", "Baba", "Hator", "Kiahk", "Toba", "Amshir", "Baramhat", "Baramouda", "Bashans", "Paona", "Epep", "Mesra"]
```
and to get the month names of the [“coptic” calendar system](https://en.wikipedia.org/wiki/Coptic_calendar)  in Traditional Chinese used in Taiwan:
```
let dn = new Intl.DisplayNames("zh-Hant-TW", {type: "month", style: "long", calendar: "coptic"})
let monthNames = [ dn.of(1), dn.of(2), dn.of(3), dn.of(4), dn.of(5), 
dn.of(6), dn.of(7), dn.of(8), dn.of(9), dn.of(10), dn.of(11), dn.of(12)]
monthNames
["1月", "2月", "3月", "4月", "5月", "6月", "7月", "8月", "9月", "10月", "11月", "12月"]
```

### TimeZone Names
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
