
<h1> Web Scraping Viewpoint.ca Data with R </h1>

<h2>About Viewpoint.ca:</h2>

The real estate listing website <a href = "viewpoint.ca">viewpoint.ca</a> has relatively complete and accurate records. The ability to query viewpoint.ca by PID (Property Identifier) makes it relatively simple to scrape for real property values/assesments in Nova Scotia.

<h2> Getting started </h2>
Load the ```tidyverse``` and ```dplyr``` libraries:

```r
library(dplyr)
library(tidyverse)
```

Load the ```Civic_Addresses.csv``` file from the <a href="https://catalogue-hrm.opendata.arcgis.com/datasets/2bc8323870fe44eab50630404713be6a_0?geometry=-64.521%2C44.422%2C-61.936%2C45.104"> Halifax Open Data Portal</a> (multiple file type options including .shp are also available). Every property lot in Canada has a unique PID. Cadastral shapefiles that show the delineated boundaries of property lots are difficult to find though. Point shapefiles with the location of the civic address and it's associated PIDs are easy to locate:

```r
addresses <- read_csv("Civic_Addresses.csv")
```
Parsing failures are due to the read_csv function interpreting the TECH_MOD column as logical, but the column values are not compliant with typically expect TRUE / FALSE values - for instance the word "GERRIOM".

Lets look at an example viewpoint page,
<a href = "https://www.viewpoint.ca/property/00599050/">"https://www.viewpoint.ca/property/00599050/"</a>

Viewpoint URLS are formatted in a simple fashion, ```/property/#####/``` where ##### is the PID (Unique Property Identifier number of the lot parcel), as shown in the URL above.

```r
viewpoint_base_url <- "https://www.viewpoint.ca/property/"
```

The column that we are going to use to correlate with the Viewpoint Data are the PIDs, we will get them from our addresses CSV file we read in earlier:

```r
#using [c("column_names_vector", "example_column")] will return a subset of columns:
addresses[c("PID")]
```

Count the number of rows in the civic address file with the ```nrow()``` function:

```r
nrow(addresses)
```

Since there may be multiple structures with the same PID, we can consider using the ```distinct()``` function from ```dplyr``` together with the ```%>%``` syntax:

```r
addresses[c("PID")] %>% distinct()
```

```r
nrow(addresses[c("PID")] %>% distinct())
```


Load the ```rvest``` library:

```r
library(rvest)
```

<h2>Retrieving the HTML Contents of a Viewpoint.ca Page:</h2>

```r
#The read_html function will retrieve the raw html content of the page:
#We'll use the first PID, #00143396 as a test:
(test_page <- read_html("https://www.viewpoint.ca/property/00143396/"))

#Note: wrapping the call in ( ) brackets makes an assignment print the contents of the assigned variable to the console
```

By analyzing the tags used by the Viewpoint developers, we'll find that the assesed property value is contained within a ```span``` tag. 
  
```r
test_page %>%
  html_nodes("span") %>% #Subset the <span> nodes
  html_text() #Get the HTML text contents within the nodes
```

We can assign this subset of the page to the existing variable, then subset just the tenth row of it to get the assessed property value:

```r
test_page <- test_page %>%
  html_nodes("span") %>% #Subset the <span> nodes
  html_text() #Get the HTML text contents within the nodes
```
Our initial results aren't particularly useful in this form, fortunately we can easily subset the HTML text contents of the vector by the "line"- i.e. row, number.

Retrieve just the row with the assessed property value using the 10th line as a test:
```r
#A single index in a vector's name followed by brackets will return that row: 
test_page[10]
```

Assigning this property value to a new test variable:
```r
test_value <- test_page[10]

test_value
```

We are interested in the change in assessed property values as well. Getting the value from this column will require some additional string manipulation to drop the unrequired text:

```r
test_page[13]
```


We can use the base R function ```gsub``` to substitue the unneeded parts of the text and leave ourselves with just the percent change and the year range:
```r
gsub('[\t\n Historical Assesment]', '', test_change)
```

<h2>Using a Loop to Iterate Through All of the PIDs and Scrape Their Assessed Value:</h2>

Constructing a basic for loop:

```r
#Create a for loop with a variable called "i" as the index that iterates from 1 through to the number of rows in the address CSV file:
for(i in 1:nrow(addresses)){
}
```

Obviously, the loop contents being empty, we will not see anything in the console, but the loop did run as many times as there are PIDs.

Adding conditions to this for loop:

```r
#Create a variable outside the loop and increment it by 1 every time the loop iterates:
counter <- 0
for(i in 1:nrow(addresses)){
  #Increment the current counter value by 1
  counter <- counter + 1
}
#Printing the counter will return the number of rows in the loop:
print(counter)
```
We could print a message every time the loop runs, but with this many iterations it would be very long. Insted we may be interested in some subset of the loop:

Adding conditions to this for loop:

```r
#Use less than  a certain number as a condition to print only once the loop runs every 10000 times:
for(i in 1:nrow(addresses)){
  if(i <= 10){ 
    print(paste("Reached: ", i, sep = ""))
  }
}
```

From the end of the loop:

```r
#Use less than  a certain number as a condition to print only once the loop runs every 10000 times:
for(i in 1:nrow(addresses)){
  if(i >= nrow(addresses)-10){ 
    print(paste("Reached: ", i, sep = ""))
  }
}
```

```r
#Use modulo as a condition to print only once the loop runs every 10000 times:
for(i in 1:nrow(addresses)){
  if(i %% 10000 == 0){ 
    print(paste("Reached: ", i, sep = ""))
  }
}
```
  
```r
#We can also combine all three together in the single loop:
for(i in 1:nrow(addresses)){
  if(i <= 10){ 
    print(paste("Reached: ", i, sep = ""))
  }
  
  if(i %% 10000 == 0){ 
    print(paste("Reached: ", i, sep = ""))
  }
  
  if(i >= nrow(addresses)-10){ 
    print(paste("Reached: ", i, sep = ""))
  }
}
```

```r
#We may also want to iteratively append the values to an position in a vector:
status_list <- ""

for(i in 1:nrow(addresses)){
  if(i <= 10){ 
    status_list[i] = paste("Reached: ", i, sep = "")
  }
  
  if(i %% 10000 == 0){ 
    status_list[i] = paste("Reached: ", i, sep = "")
  }
  
  if(i >= nrow(addresses)-10){ 
    status_list[i] = paste("Reached: ", i, sep = "")
  }
}

#Using is.na() to get the indexes of the NA, values, we can print only the locations where the value is not NA, ie the position we wrote to earlier.
status_list[!is.na(status_list)]
```

<h2>Anyways, back to business:</h2>

```r
addresses %>% mutate("PROP_VAL")
```

To test how Viewpoint reacts to being rapidly queried, I'll try just 10 requests to start with:
```r
for(i in 1:10){
  cur_URL <- (paste(viewpoint_base_url, addresses[i,4], "/", sep =""))
  print(cur_URL)
}
```
Seems pretty fast. 

Lets scrape the whole site:
```r
scraped_values <- matrix(NA, 0, 3)
scraped_values <- colnames(c("PID","Value","Change"))

print(paste("items to scrape: ",nrow(addresses)))

for(i in 1:nrow(addresses)){
  
  cur_URL <- (paste(viewpoint_base_url, addresses[i,4], sep =""))
  print(paste("scraping item :", i, " ", cur_URL));

  cur_page <- read_html(cur_URL)
  
  cur_page <- cur_page %>%
    html_nodes("span") %>% #Subset the <span> nodes
    html_text() #Get the HTML text contents within the nodes
  
  cur_scrape <- gsub('[\t\n]', '', cur_page[13])
  cur_scrape <- gsub('[Assesment,&,Tax]', '', cur_scrape)
  cur_scrape <- gsub('.*? ', '', cur_scrape)
  cur_scrape <- strsplit(cur_scrape, '\\$', fixed=t)
  cur_value <- cur_scrape[[1]][2]
  cur_change <- cur_scrape[[1]][3]

  scraped_values <- rbind(c(addresses[i,4], cur_value, cur_change), scraped_values);

  Sys.sleep(.05)
}

scraped_values
```
