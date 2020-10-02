# Data Scraping in R Using `rvest`

<a href="http://creativecommons.org/licenses/by-nc/4.0/" rel="license"><img style="border-width: 0;" src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png" alt="Creative Commons License" /></a>
This tutorial is licensed under a <a href="http://creativecommons.org/licenses/by-nc/4.0/" rel="license">Creative Commons Attribution-NonCommercial 4.0 International License</a>.

## Lab Objectives

This lab covers scraping data from the web into R using the `rvest` package. 

Web pages are written in `HTML (Hyper Text Markup Language)` which uses `tags` to describe different aspects of document content. For example, a heading in a document is indicated by `<h1>My Title</h1>` whereas a paragraph would be indicated by `<p>A paragraph of content...</p>`.

In this tutorial, we will learn how to read data from a table on a web page into R. We will need the package `rvest` to get the data from the web page, and the `stringr` package to clean up the data.

As described in the package documentation, `rvest` helps you scrape information from web pages, specifically making it easier to scrape (or harvest) data from html web pages. `rvest` works with `magrittr` so that you can express complex operations as elegant pipelines composed of simple, easily understood pieces. 

For more details on the `rvest` package as part of the `tidyverse`, check out the [GitHub package documentation](https://github.com/tidyverse/rvest) or a [Hadley Wickham blog post about `rvest`](https://blog.rstudio.com/2014/11/24/rvest-easy-web-scraping-with-r/).

## Acknowledgements

This lab is based on the ["Introduction to Data Scraping with R"](https://remiller1450.github.io/s230s19/Intro_to_Web_Scraping.html) lab materials developed by [Dr. Ryan Miller](https://remiller1450.github.io/) for the the [STA 230 "Introduction to Data Science" course](https://remiller1450.github.io/sta230f19.html). 

# Table of Contents
- [Setting Up the Environment](#setting-up-the-environment)
- [Scraping a Wikipedia Table Using `rvest`](#scraping-a-wikipedia-table-using-rvest)
- [Scraping Movie Box Office Data](#scraping-movie-box-office-data)
- [Parsing Unstructured Text Using HTML and CSS](#parsing-unstructured-text-using-html-and-css)
- [Lab Notebook Questions](#lab-notebook-questions)

# Setting Up the Environment

1. Packages we'll need for this lab: `rvest`, `stringr`, `readr`.

2. To install, copy and run in console or uncomment and run in your R script.
```R
# install.packages("rvest")
# install.packages("stringr")
# install.packages("readr")
```

3. Load the packages using the `library()` command.
```R
library(rvest)
library(stringr)
library(readr)

```

4. Alternate approach: Want to try installing and loading multiple packages simultaneously? List multiple packages within the same `install.packages()` or `library()` command.
```R
# install.packages("rvest", "stringr", "readr")
# library(rvest, stringr, readr)
```

# Scraping a Wikipedia Table Using `rvest`

5. Navigate to the https://en.wikipedia.org/wiki/List_of_countries_by_population_in_1900 URL in a web browser. 

6. What tabular data do you see on the page? (i.e. is there a table?)

7. Read the web page into R using the `read_html` command from the `rvest` package. `read_html` uses HTML's markup language structure and tags to parse the web page into an R object.
```R
popParse <- read_html("https://en.wikipedia.org/wiki/List_of_countries_by_population_in_1900")
str(popParse)
```
8. `read_html` creates an R object with information about the web page, based on the HTML tags present in the web page.

9. We now have a `popParse` object that includes nodes, or HTML elements from the web page.

<blockquote>Why are we talking about markup language, HTML, and tags? Most web page are written using hyper-text markup language (HTML). HTML is a type of markup language, that is based on tags <code><tag></tag></code> that "mark up" different elements of text and shape how they display on a web page. HTML syntax involves opening and closing tags (example: <code><tag></tag></code>) that don't appear on the web page but are part of the back-end structure of the web page. Looking for patterns in these tags can help us identify and parse different parts of a web page. For more on HTML, visit <a href="https://www.w3schools.com/html/html_intro.asp">W3Schools HTML Introduction</a>.</blockquote>

10. We can now work on transforming our `popParse` object into a table. This will involve multiple steps.

11. First step is to see what tables are in the `popParse` object.
```R
popNodes <- html_nodes(popParse, "table")
popNodes
```

12. `popNodes` gives us a listing of the tables in the `popParse` object. 

13. We can select one of those tables and extract it as a new object.

14. We'll do this by selecting the fourth table using double brackets, which are used to index list object.
```R
# creates a new object pop with the content of the sixth table from popNodes
pop <- html_table(popNodes, header = TRUE, fill = TRUE)[[6]] 
str(pop)
```

15. What is an index list? An index list is a list of the items in a list based on their position. 

16. For example, in the list `apple, pear, banana, peach`, `apple` is in the first (`[1]`) index position, `pear` is `[2]`, and so on.

<blockquote>NOTE: In the R/RStudio scripting language, index position counting starts at one (1). This is different than other programming languages like Python where counting starts at zero (0).</blockquote>

17. Now we can start to focus on cleaning the `pop` data frame.

18. Notice that even though the first and third columns are numbers, they are classified as “character.” 

19. For Rank, that is because the first observation is the world population and it is not assigned a rank, but rather, the character “-”. 

20. We can remove the world population row.
```R
# creates new pop2 object with first row removed from pop object
pop2 <- pop[-1, ]
head(pop2)
```

21. We will need to reset the row numbers.
```R
# reset row numbers
row.names(pop2) <- NULL
```

22. We're still stuck with Rank and Population columns that are not numbers.

23. We need to convert these columns to be numeric.
```R
# force rank field to be a number
# this output WILL include an error message--don't panic!
pop2$Rank <- as.numeric(pop2$Rank)
```

24. The Population column is also a character because the numbers have commas in them, plus some observations include characters such as [1] to indicate some footnotes.

25. We can use the `parse_number` command to remove commas and footnotes from the Population column.
```R
# remove commas and footnotes from population column
pop2$Population <- parse_number(pop2$Population)
```

26. We can also rename the third column to `Population`.
```R
names(pop2)[3] <- "Population"
```

27. Almost there! We can use regular expressions to remove footnotes from the `Country/Territory` field.
```R
pop2$`Country/Territory` <- str_replace_all(pop2$`Country/Territory`, "\\[[^]]+\\]", "")
head(pop2)
```

28. Take a look at `pop2`. Now we have a data frame that could be used for different kinds of analysis and visualization.

# Scraping Movie Box Office Data

29. The web site [Box Office Mojo](http://www.boxofficemojo.com/) gives statistics on box office earnings of movies. In addition to daily earnings, the web site also maintains lists of yearly and all time record holders.

30. We will look at the movies in the top 100 in all time movie worldwide grosses in box office receipts. 

31. Specifically, we'll scrape the data from [Box Office Mojo: All Time Box Office](http://www.boxofficemojo.com/alltime/world/?pagenum=1).

32. Open the http://www.boxofficemojo.com/alltime/world/?pagenum=1 web page in a browser. 

33. What tabular data do you see on the page? (i.e. is there a table?)

34. First step is using `read_html` to parse the data.
```R
# read in box office mojo data about box office receipts
movieParse<- read_html("http://www.boxofficemojo.com/alltime/world/?pagenum=1")
```

35. Now we want to create a data object from the `movieParse` nodes.
```R
movieTables <- html_nodes(movieParse, "table")
head(movieTables)
```

36. We want to take the `movieTables` data object and extract table data.
```R
movies <- html_table(movieTables, header = TRUE, fill = TRUE)[[1]]
str(movies)
```

37. For the `Lifetime Gross` field to be a number, we need to use regular expressions to remove the dollar signs and commas from that field.
```R
out <- str_replace_all(movies$`Lifetime Gross`, "\\$|,", "" )
head(out)
```

38. Now we want to convert the `Lifetime Gross` field to numbers.
```R
movies$`Lifetime Gross` <- as.numeric(out)
```

39. So far the two examples we’ve looked at extract tables from a webpage, but lots of other information from a webpage can be extracted using `rvest`. 

40. For example, we could extract image data from the Box Office Mojo webpage.
```R
moviesImg <- html_nodes(movieParse, "img")
head(moviesImg)
```

# Parsing Unstructured Text Using HTML and CSS

41. This section of the lab will look at David Leonhardt and Stuart A. Thompson's December 2017 *New York Times* article "[Trump's Lies](https://www.nytimes.com/interactive/2017/06/23/opinion/trumps-lies.html)."

42. Our goal here is to transform the body of text into a clean data.frame containing three fields:
- Text of the lie
- Lie date
- Lie URL source

43. Navigate to https://www.nytimes.com/interactive/2017/06/23/opinion/trumps-lies.html in a web brwoser. Explore the public-facing page as well as the source HTML.

44. Sometimes we might want to extract specific nodes that are defined by CSS (Cascading Style Sheets, a langauge which defines the style of an html page). For this goal, it can be very useful to view to html source code of a page for guidance.

45. To view the HTML code behind a webpage, right click anywhere on the page and select “View Page Source” in Chrome or Firefox, “View Source” in Internet Explorer, or “Show Page Source” in Safari. (If that option doesn’t appear in Safari, just open Safari Preferences, select the Advanced tab, and check “Show Develop menu in menu bar”.)

46. From inspecting the html source code, we see that each lie has the following format.
```HTML
<span class="short-desc"><strong> DATE </strong> LIE <span class="short-truth"><a href="URL"> EXPLANATION </a></span></span>
```

47. This tells us that extracting all `<span>` tags belonging to the class `“short-desc”` will provide us what we need.

48. First step is creating a data object from the parsed HTML.
```R
# reads HTML
NytTrump <- read_html("https://www.nytimes.com/interactive/2017/06/23/opinion/trumps-lies.html")
```

49. Then we want to extract the nodes that include the `"short-desc"` class.
```R
# extracts short-desc nodes 
lies <- html_nodes(NytTrump, ".short-desc")
lies
```

50. The `“.”` in from of `“short-desc”` is [CSS selector syntax](https://www.w3schools.com/cssref/css_selectors.asp), it will select all elements with `class=“short-desc”`. See [CSS selector link documentation from W3Schools](https://www.w3schools.com/cssref/css_selectors.asp) for more selector examples.

51. Next step is to find the date of each lie. Again we're looking for patterns in the HTML tags that let us see and find the elements we're looking for.

52. For this we notice that each date is wrapped within a `<strong>` tag. Thus, extracting nodes with the `<strong>` tag will provide the dates we need
```R
# extracting nodes with <strong> tag
dates <- html_nodes(lies, "strong")
```

53. We also want to remove non-text content from our `dates` object.
```R
# removes non-text content using html_text
dates <- html_text(dates, trim = TRUE)
head(dates)
```

54. Extracting each lie requires a different strategy. We could do it using regular expressions, but instead we use the `xml_contents` function.
```R
# extract each lie (could also do this using regular expressions)
lies_var <- xml_contents(lies)
head(lies_var)
```

55. We see that each lie has three components, the second containing the lie itself, and these components are arranged in a vector. 

56. We can exploit this ordering and extract the lies using their positions.
```R
# each lie has three components
# lie is the second component
# we can use the ordering and position to extract the lies
lies_var2 <- lies_var[seq(2, length(lies_var), by = 3)]
head(lies_var2)
```

57. We can use a similar strategy to get the urls.
```R
# gets the URLs for each lie
lies_var3 <- lies_var[seq(3, length(lies_var), by = 3)]
head(lies_var3)
```

58. From here we see that all the urls are tagged with `<a href...>` tag. 

59. We can select these nodes using the tag, and then select the `href` object to get the url itself. 
```R
# <a href...> tag lets us isolate just the URLs
lies_var3 <- html_node(lies_var3, "a")
url_var <- html_attr(lies_var3, "href")
head(url_var)
```

60. At this point we’ve met our original goal. But a next step might be a textual analysis that further extracts key words or categorizes the urls.
```R
# extracting key words or categorizing URLs
lies_clean <- data.frame(date = dates, lie = as.character(lies_var2), url = url_var)
head(lies_clean)
```

# Lab Notebook Questions

Q1: Take a data table from a web page of your choosing and convert to an R data frame. Include comments that document steps you're taking in that process.

Q2: Describe how you approached Q1. What problems did you encounter? How did you solve them?

Q3: Describe what kinds of analysis or visualization you could do with this data. What steps would you take or incorporate to reflect data feminism principles/values?

Q4: Take unstructured text on a web page and convert it to an R data frame. Data frame could include multiple fields (like the lab's NYT example) or include unstructured text.

Q5: Describe how you approached Q2. What problems did you encounter? How did you solve them?

Q6: Describe what kinds of analysis or visualization you could do with this data.  What steps would you take or incorporate to reflect data feminism principles/values?
