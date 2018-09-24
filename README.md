
[![Travis build
status](https://travis-ci.org/ATFutures/ical.svg?branch=master)](https://travis-ci.org/ATFutures/ical)

<!-- README.md is generated from README.Rmd. Please edit that file -->

# ical

The goal of ical is to work with iCalander (`.ics`, `.ical` or similar)
files in R. iCalendar is an open standard for “exchanging calendar and
scheduling information between users and computers” described at
[icalendar.org](https://icalendar.org/) (the full spec can be found in a
plain text file [here](https://tools.ietf.org/rfc/rfc5545.txt)).

Recently the UK Government endorsed the iCal format in a
[publication](https://www.gov.uk/government/publications/open-standards-for-government/exchange-of-calendar-events)
for the ‘Open Standards for Government’ series. [An example .ics
file](https://www.gov.uk/bank-holidays/england-and-wales.ics) is
provided by the .gov.uk domain, which shows holidays in England and
Wales.

## Installation

``` r
devtools::install_github("ATFutures/ical")
#> Using GitHub PAT from envvar GITHUB_PAT
#> Skipping install of 'ical' from a github remote, the SHA1 (1e4caa6b) has not changed since last install.
#>   Use `force = TRUE` to force installation
```

``` r
library(ical)
```

<!-- You can install the released version of ical from [CRAN](https://CRAN.R-project.org) with: -->

<!-- ``` r -->

<!-- install.packages("ical") -->

<!-- ``` -->

## Example

A minimal example representing the contents of an iCalendar file is
provided in the dataset `ical_example`, which is loaded when the package
is attached. This is what iCal files look like:

``` r
ical_example
#>  [1] "BEGIN:VCALENDAR"                                  
#>  [2] "PRODID:-//Google Inc//Google Calendar 70.9054//EN"
#>  [3] "VERSION:2.0"                                      
#>  [4] "CALSCALE:GREGORIAN"                               
#>  [5] "METHOD:PUBLISH"                                   
#>  [6] "X-WR-CALNAME:atf-test"                            
#>  [7] "X-WR-TIMEZONE:Europe/London"                      
#>  [8] "BEGIN:VEVENT"                                     
#>  [9] "DTSTART:20180809T160000Z"                         
#> [10] "DTEND:20180809T163000Z"                           
#> [11] "DTSTAMP:20180810T094100Z"                         
#> [12] "UID:1119ejg4vug5758527atjcrqj3@google.com"        
#> [13] "CREATED:20180807T133712Z"                         
#> [14] "DESCRIPTION:\\n"                                  
#> [15] "LAST-MODIFIED:20180807T133712Z"                   
#> [16] "LOCATION:"                                        
#> [17] "SEQUENCE:0"                                       
#> [18] "STATUS:CONFIRMED"                                 
#> [19] "SUMMARY:ical programming mission"                 
#> [20] "TRANSP:OPAQUE"                                    
#> [21] "END:VEVENT"                                       
#> [22] "END:VCALENDAR"
```

Relevant fields can be found and extracted as follows:

``` r
ic_find(ical_example, "TSTAMP")
#>  [1] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE  TRUE
#> [12] FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE FALSE
ic_extract(ical_example, "TSTAMP")
```

A larger example shows all national holidays in England and Wales. It
can be read-in as
follows:

``` r
ics_file <- system.file("extdata", "england-and-wales.ics", package = "ical")
ics_raw = readLines(ics_file) 
head(ics_raw) # check it's in the ICS format
#> [1] "BEGIN:VCALENDAR"                     
#> [2] "VERSION:2.0"                         
#> [3] "METHOD:PUBLISH"                      
#> [4] "PRODID:-//uk.gov/GOVUK calendars//EN"
#> [5] "CALSCALE:GREGORIAN"                  
#> [6] "BEGIN:VEVENT"
```

A list representation of the calendar can be created using `ic_list()`
as follows:

``` r
ics_list = ic_list(ics_raw)
ics_list[1:2]
#> [[1]]
#> [1] "DTEND;VALUE=DATE:20120103"                    
#> [2] "DTSTART;VALUE=DATE:20120102"                  
#> [3] "SUMMARY:New Year’s Day"                       
#> [4] "UID:ca6af7456b0088abad9a69f9f620f5ac-0@gov.uk"
#> [5] "SEQUENCE:0"                                   
#> [6] "DTSTAMP:20180806T114130Z"                     
#> 
#> [[2]]
#> [1] "DTEND;VALUE=DATE:20120407"                    
#> [2] "DTSTART;VALUE=DATE:20120406"                  
#> [3] "SUMMARY:Good Friday"                          
#> [4] "UID:ca6af7456b0088abad9a69f9f620f5ac-1@gov.uk"
#> [5] "SEQUENCE:0"                                   
#> [6] "DTSTAMP:20180806T114130Z"
```

A data frame representing the calendar can be created as follows (work
in progress):

``` r
ics_df = ic_read(ics_file) # read it in
head(ics_df) # check the results
#> # A tibble: 6 x 6
#>   `DTEND;VALUE=DAT… `DTSTART;VALUE=D… SUMMARY  UID       SEQUENCE DTSTAMP 
#>   <date>            <date>            <chr>    <chr>     <chr>    <chr>   
#> 1 2012-01-03        2012-01-02        New Yea… ca6af745… 0        2018080…
#> 2 2012-04-07        2012-04-06        Good Fr… ca6af745… 0        2018080…
#> 3 2012-04-10        2012-04-09        Easter … ca6af745… 0        2018080…
#> 4 2012-05-08        2012-05-07        Early M… ca6af745… 0        2018080…
#> 5 2012-06-05        2012-06-04        Spring … ca6af745… 0        2018080…
#> 6 2012-06-06        2012-06-05        Queen’s… ca6af745… 0        2018080…
```

What class is each column?

``` r
vapply(ics_df, class, character(1))
#>   DTEND;VALUE=DATE DTSTART;VALUE=DATE            SUMMARY 
#>             "Date"             "Date"        "character" 
#>                UID           SEQUENCE            DTSTAMP 
#>        "character"        "character"        "character"
```

## Trying on calendars ‘in the wild’

To make the package robust we test on a wide range of ical formats.
Here’s an example from my work calendar, for example:

``` r
my_cal = ic_dataframe(ical_outlook)
my_cal$SUMMARY[1]
#> [1] "In Budapest for European R Users Meeting (eRum) conference"
my_cal$DTSTART[1]
#> NULL
my_cal$DTEND[1]
#> NULL
my_cal$DTEND[1] - my_cal$DTSTART[1]
#> integer(0)
```

## Related projects

  - A Python package for working with ics files:
    <https://github.com/C4ptainCrunch/ics.py>
  - A JavaScript package by Mozilla:
    <https://github.com/mozilla-comm/ical.js/>
