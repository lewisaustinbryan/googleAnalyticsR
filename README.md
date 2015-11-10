# googleAnalyticsR
A new Google Analytics R library.  Very much work in progress, mainly here to test implementation of a Google API package using `googleAuthR`

## Other GA R libraries

* [rga](https://github.com/skardhamar/rga)
* [RGA](https://bitbucket.org/unikum/rga)
* [RGoogleAnalytics](https://github.com/Tatvic/RGoogleAnalytics)
* [ganalytics](https://github.com/jdeboer/ganalytics)
* [GAR](https://github.com/andrewgeisler/GAR)

## Why do we need another GA library?

This one uses googleAuthR as a backend, which means:  
* Shiny App compatible
* The same authentication flow can be used with other googleAuthR apps like searchConsoleR
* Automatic batching, sampling avoidance with daily walk, multi-account fetching, multi-channel funnnel
* Support for googleAuthR batch, meaning 10 calls at once to GA - great for big fetches.  For big data calls this is implemented automatically so could be 10x quicker than normal GA fetching

* Meta data included in attributes of returned dataframe

## Install

Development version

```
devtools::install_github("MarkEdmondson1234/googleAuthR")
devtools::install_github("MarkEdmondson1234/googleAnalyticsR_public")
```
## To use

```
library(googleAuthR)

library(googleAnalytics)
=======
library(googleAnalytics_public)


## set options if needed
options("googleAuthR.scopes.selected" = getOption("googleAnalyticsR.scope") )
options("googleAuthR.client_id" = getOption("googleAnalyticsR.client_id"))
options("googleAuthR.client_secret" = getOption("googleAnalyticsR.client_secret"))
options("googleAuthR.webapp.client_id" = getOption("googleAnalyticsR.webapp.client_id"))
options("googleAuthR.webapp.client_secret" = getOption("googleAnalyticsR.webapp.client_secret"))

gar_auth()

## get account info, including View Ids
account_list <- google_analytics_account_list()

## get a list of what metrics and dimensions you can use
meta <- google_analytics_meta()
head(meta)

## pick the account_list$viewId you want to see data for.
## metrics and dimensions can have or have not "ga:" prefix
gadata <- google_analytics(account_list$viewId[1], 
                           start="2015-08-01", end="2015-08-02", 
                           metrics = c("sessions", "bounceRate"), 
                           dimensions = c("source", "medium"))

## multi accounts, pass character vector of viewIds
multi_gadata <- google_analytics(account_list$viewId[1:2], 
                                 start="2015-08-01", end="2015-08-02", 
                                 metrics = c("sessions", "bounceRate"), 
                                 dimensions = c("source", "medium"))

## if more than 10000 rows in results, auto batching
batch_gadata <- google_analytics(account_list$viewId[1], 
                                 start="2014-08-01", end="2015-08-02", 
                                 metrics = c("sessions", "bounceRate"), 
                                 dimensions = c("source", "medium", "landingPagePath"))

## mitigate sampling by setting samplingLevel="WALK"
walk_gadata <- google_analytics(account_list$viewId[1], 
                                start="2014-08-01", end="2015-08-02", 
                                metrics = c("sessions", "bounceRate"), 
                                dimensions = c("source", "medium", "landingPagePath"), 
                                samplingLevel="WALK")

## multi-channel funnels set type="mcf"
mcf_gadata <- google_analytics(account_list$viewId[1], 
                               start="2015-08-01", end="2015-08-02", 
                               metrics = c("totalConversions"), 
                               dimensions = c("sourcePath"), 
                               type="mcf")

## google_analytics dataframes include these meta data attributes:
- attr(*, "containsSampledData")= logi FALSE
 - attr(*, "samplePercent")= num 100
 - attr(*, "samplingLevel")= chr "DEFAULT"
 - attr(*, "profileInfo")=List of 6
  ..$ profileId            : chr "535656"
  ..$ accountId            : chr "2748374"
  ..$ webPropertyId        : chr "UA-278374-1"
  ..$ internalWebPropertyId: chr "5279208"
  ..$ profileName          : chr "XXXXX"
  ..$ tableId              : chr "mcf:539656"
 - attr(*, "dateRange")=List of 2
  ..$ startDate: chr "2015-08-01"
  ..$ endDate  : chr "2015-08-02"
 - attr(*, "totalResults")= int 4

## reach meta-data via attr()
attr(gadata, "profileInfo")
attr(gadata, "dateRange")

```

To use in Shiny, use the googleAuth `with_shiny`

```

## in server.R
library(googleAuthR)
library(googleAnalyticsR)
library(shiny)

shinyServer(function(input, output, session){
  
  ## Get auth code from return URL
  access_token  <- reactiveAccessToken(session)
  
  ## Make a loginButton to display using loginOutput
  output$loginButton <- renderLogin(session, access_token(),
                                    logout_class = "btn btn-danger")

  gadata <- reactive({

    gadata <- with_shiny(google_analytics,
                         id = "222222",
                         start="2015-08-01", end="2015-08-02", 
                         metrics = c("sessions", "bounceRate"), 
                         dimensions = c("source", "medium"),
                         shiny_access_token = access_token())

})
```