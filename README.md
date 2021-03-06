# Guide and tips to use SIE API

Banco de Mexico has developed an API for Developers, Analysts and Researchers to automatically retrieve the databases from the SIE. In this repository I share tips and tools to use this useful API with R.

```diff
# ⚠ The views and conclusions presented in this Repository are exclusively   # 
#     the responsibility of the author and do not necessarily reflect those  #
#     of Banco de Mexico.                                                    #
```

[![License:
MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

## Overview
Central Banks are some of the institutions with the best reputation in each country, with the main aims of control the Inflation, supervise the Financial Institutions, and produce and distribute Currency at a national level, among other tasks.

[Banco de Mexico](https://www.banxico.org.mx/indexen.html), founded in 1925 and autonomy since 1994, plays a key role in the Economy of the Mexico. As the Central Bank, it publishes the archive of national economic databases through the [Economic Information System](https://www.banxico.org.mx/SieInternet/defaultEnglish.do), known as __SIE__.  

Additionally, this Central Bank has developed an API that allows Developers, Analysts and Researchers to consult automatically the SIE's time series, using JavaScript, Java, C# and R.  More details of the API can be found in the [SIE API official webpage](https://www.banxico.org.mx/SieAPIRest/service/v1/?locale=en).  

The aim of this repository is to show __how to use SIE's API with R__, explaining how to retrieve information with the `siebanxicor` R package developed by Banco de Mexico, and how to use a custom function to explore the selected time series, and a Dashboard to look into Banknotes and Coins series.

## Getting prepared
As mentioned before, in this repository we will focus in how to use SIE API with R.  

### Request a token
The first step is to get a Bmx-Token. You can request a 64-alphanumeric token in [this link](https://www.banxico.org.mx/SieAPIRest/service/v1/token?locale=en), which would be required to use the API.  

### Installation of R package
Banco de Mexico developed the `siebanxicor` R package to automatically retrieve economic databases published in the SIE by Banco de Mexico. You can install it by running the next line in R or Rstudio:
```{r}
install.packages("siebanxicor")
```

More details of `siebanxicor` can be found [⭐️ here](https://cran.r-project.org/web/packages/siebanxicor/siebanxicor.pdf).

## Features of `siebanxicor`
The `siebanxicor` R package has five utility functions to retrieve information from the databases published by the Mexican Central Bank: 

- `setToken` - while the Bnx-Token is the key to access SIE API, the `setToken` function is the knob that opens the channel and connects to this API. This utility function <u>should be called before any other function</u> from the siebanxicor package.  

- `getSeriesData` - requests the time series data from the SIE, up to 100 series at a time. It returns a vector with the requested information.  

- `getSeriesMetadata` - retrieves [metadata](https://en.wikipedia.org/wiki/Metadata) for the requested series.  

- `getSerieDataFrame` - allows to get a data.frame from <u>only one</u> time series of the vector returned by the `getSeriesData` utility function.  

- `getSeriesCurrentValue` - requests the time series data from the SIE, up to 100 series at a time, returning a data.frame with only the <u>last value</u> of the requests time series.  

_Note: In some cases, to use one of these functions you should previously use another(s) of the mentioned functions; for example, to use `getSeriesMetadata` you should previously call `setToken` and `getSeriesData`._  

<br>

Additionally, I have created a custom support function that can be found in the __src__ folder of this repo, to help Analysts and Researchers to easily explore the time series:  

- `sie_function` - when giving the names of the code of different series, it automatically prints and saves the plot of this series, prints the metadata, and returns the data in a [tidy format](https://www.jstatsoft.org/article/view/v059i10) frame format.  

To complement this effort, I built a [Shiny Dashboard](https://github.com/vcuspinera/Banxico_SIE_API_guide/tree/main/sie_app) that focus on Mexican Banknotes and Coins exploring currency time series from SIE API with the option of saving the database of the selected series, in wide or tidy format.

## Dependencies

|R packages |
|:----------|
|siebanxicor|
|tidyverse  |
|tidyr      |
|reshape2   |
|reticulate |
|shiny      |
|shinydashboard|
|shinyWidgets|

## Usage

To show the usage of the SIE API with `siebanxicor` R package, we will go through an example using the time series for the of the __Annual counterfeit domestic banknotes detected for the current denomination in circulation__: 20, 50, 100, 200, 500 and 1000 pesos  (series __SM1249__, __SM1250__, __SM1251__, __SM1252__, __SM1253__, __SM1254__).

[⭐️ Click here](https://www.banxico.org.mx/SieAPIRest/service/v1/doc/catalogoSeries?locale=en) to look for the complete catalogue of the SIE's time series published by Banco de Mexico.

### Utility functions of `siebanxicor`

#### 1. Load library

After the `siebanxicor` package is installed, load this library and also the tidyverse library.
```{r load libraries}
library("siebanxicor")
library("tidyverse")
```

#### 2. Use `setToken(token)`

Bring your token and open the SIE API channel with the `setToken` utility function.
```{r bring and set token, warning=FALSE}
# bring the token
token_file <- read.csv("../token/SIE_Token.csv", header=FALSE)

# set the token
setToken(token_file$V2)
```
_Notes:_  
- _If you don't have a token to use SIE API, ⭐️ [**click here**](https://www.banxico.org.mx/SieAPIRest/service/v1/token) to access the official website and obtain one._  
- _I add a csv file where users should paste and save their token to run this example._  


#### 3. Get data with `getSeriesData(series, startDate, endDate)`

Get the time series of interest, in this case the series of the annual counterfeit of mexican banknotes per denomination, using the `getSeriesData` function.
```{r getting series}
# setting the variables
my_series <- c("SM1249", "SM1250", "SM1251", "SM1252", "SM1253", "SM1254")
my_start <- '2015-01-01'
my_end <- Sys.Date() #looks for today's date

# getting the series
series <- getSeriesData(my_series, my_start, my_end)
```
_Note: to use the `getSeriesData` function, you should previously call `setToken`._


#### 4. Get the metadata with `getSeriesMetadata(series, locale)`

This function returns the general information of series. To select the language of the metadata, set the _locale_ variable as "en" for English, and "es" for Spanish.

```{r getting metadata}
# getting the metadata
metadata <- getSeriesMetadata(my_series, locale="en")
```

_Note: to use the `getSeriesMetadata` function, you should previously call `setToken`._


#### 5. Get a data frame of one series using `getSerieDataFrame(series, idSerie)`

This function will be helpful to get a data frame for the annual counterfeit number of 500 pesos banknotes (__SM1253__ series), from the vector returned by the `getSerieDataFrame` in the previous point #3.

```{r get SM1255 df}
# getting the series
df_SM1255 <- getSerieDataFrame(series, "SM1255")
```
_Note: to use the `getSerieDataFrame` function, you should previously call `setToken` and `getSerieData`._


#### 6. Get the last value of one or more series with `getSeriesCurrentValue(series)`

To get the last value of the annual counterfeit banknotes per denomination series, we will use the `getSeriesCurrentValue` function.

```{r last value}
df_last_value <- getSeriesCurrentValue(my_series)
```
_Note: to use the `getSeriesCurrentValue` function, you should previously call `setToken`._

### Custome resources

#### 7. Use the custom function `sie_function(series_codes, series_names, title_plot, x_lab, y_lab, startDate, endDate, route)`

This function prints the metadata for the Annual Counterfit Banknotes series, prints and saves and their plot, and returns a data frame with these series in tidy format.

```{r custome function}
# call the customed function from an RScript
source("SIE_function.R")

# setting the variables
my_series <- c("SM1249", "SM1250", "SM1251", "SM1252", "SM1253", "SM1254")
my_names <- c("20", "50", "100", "200", "500", "1000")
my_title <- "Annual Counterfeit Banknotes per Denomination"
my_start <- '2010-01-01'
my_end <- Sys.Date() #looks for today's date

# run the function
df <- sie_function(my_series, my_names, my_title, route="../img/",
             y_lab="Number of Pieces", x_lab="Year", startDate=my_start)
```

In this case, the function plots the series for the Annual Counterfeit of Banknotes per denomination as follows.

<center><img class="center" src="img\Annual Counterfeit Banknotes per Denomination.png" width="65%" height="65%" class="center"></center>

⭐️ [Click here](https://github.com/vcuspinera/Banxico_SIE_API_guide/blob/main/src/SIE_function_examples.pdf) to access to a complementary document developed to show additional examples of this custome function, applied to different contexts with time-series published in the SIE by Banco de Mexico.

#### 8. Dashboard

The goal of this dashboard is to help users to easily explore time series from SIE API. In this case, the app is focus on currency series, but it could be expanded in the future.

It includes different options to select up to 10 series, in a given date range, with the possibility of changing the title and label of the plot, select a format to present the database, and a button to download the series' database in the select format.

<center><img src="img\sie_app - sketch.png" width="100%" height="80%"></center>

There are different options to run this app:

- __App file__: Open the file `app.R` located in the [sie_app folder](https://github.com/vcuspinera/Banxico_SIE_API_guide/tree/main/sie_app), and click on `▶️ Run App`.

- __R project__: Open the R project of this repository named as `SIE_API_guide.Rproj`, and run in the console:
```r
shiny::runApp("sie_app")
```

- __Terminal__: Open the terminal and, from the main folder of this repository, run the following line:
```
Rscript -e 'library(methods); shiny::runApp("sie_app/", launch.browser = TRUE)'
```

## License
If you use Banco de Mexico's SIE API, you must clearly state the source and include a reference to Banco de México's URL address to enable third parties to verify the information's accuracy. For more details, look into the [Disclaimer](https://www.banxico.org.mx/footer-en/disclaimer-usage-policies-ban.html) shared by the Central Bank.

Additionally, the materials of this repository are of public domain. If re-using/re-mixing the material, please provide attribution and link to the repository webpage.

## References

<div id="refs" class="references">

<div id="ref-sie">
Banco de Mexico. 2020. Economic Information System (SIE).
<https://www.banxico.org.mx/SieInternet/defaultEnglish.do>
</div>
<br>
<div id="ref-sie_api">
Banco de Mexico. 2020. SIE API.
<https://www.banxico.org.mx/SieAPIRest/service/v1/?locale=en>
</div>
<br>
<div id="ref-tidydata">
Wickham, Hadley. 2014. "Tidy Data". Journal of Statistical Software, Articles. Volume 59, number 10. DOI: 10.18637/jss.v059.i10
<https://www.jstatsoft.org/article/view/v059i10>
</div>
<br>
<div id="ref-shiny">
RStudio. 2020. Shiny: web application framework for R. R package, version 1.5.0
<https://shiny.rstudio.com/tutorial/>
</div>

</div>
