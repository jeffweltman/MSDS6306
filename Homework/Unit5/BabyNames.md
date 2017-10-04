##########################################################################################################################################
## Jeff Weltman                                                                                                                          #
## 10/3/2017                                                                                                                             #
## MSDS 6306-401 Unit 5 - Baby Names Assignment: Raw Data to Merged Tidy Data                                                            #                        
##########################################################################################################################################

### This assignment presents the scenario wherein we are hired to discern recently popular baby names to assist our client in the naming process.

## Raw Data Files
* yob2015.txt - This text file is comma separated, contains no headers, and includes the data for childn's name, child's gender (binary M or F), and number of times this name was given to a baby born in 2015 in the United States. Source of the data file is unknown.
* yob2016.txt - This text file is **semicolon** separated, contains no headers, and includes the data for child's name, child's gender (binary M or F), and number of times this name was given to a baby born in 2016 in the United States. Source of the data file is unknown.

### B

## Data Munging - yob2016.txt

### First, per the instructions, **yob2016.txt** was ingested.
df <- read.table("yob2016.txt", sep=";",header=FALSE,col.names=c("Name","Gender","Number_Of_Uses"))

### *Note that the column mames of **Name**, **Gender**, and **Number_Of_Uses** have been assigned to this new data frame for human readability.*

summary(df)

### We can see that there are 32,869 rows across the three new columns.

### Our client informs us that one name was erroneously and reduntantly entered as ending with three y's. We identify that row and remove it with the following code:

grep("yyy$",df$Name)
y2016 <- df[-212,]

### Note now that we have a new data frame, **y2016**, which consists of the three columns as named above and now 32,868 rows.

## Data Munging - yob2015.txt

### We ingest yob2015.txt with the following code, noting that it is comma separated. The same column names are assigned for ease of merging and readability.

y2015 <- read.table("yob2015.txt", sep=",",header=FALSE,col.names=c("Name","Gender","Number_Of_Uses"))

### y2015 is a data frame which contains 33,063 rows across the three new columns.
### No errors are apparent in this file, so we may proceed with merging.

## Merging Our Data Files

### To simplify, we want to consolidate the Name and Gender columns but maintain distinct **Number_Of_Uses** columns for our next step of discerning a sum. 

final <- merge(y2016, y2015, union("Name","Gender"), all=FALSE)

### This yields a new data frame, **final**, with four columns:
* Name
* Gender
* Number_Of_Uses.x
* Number_Of_Uses.y

### As we do not want any NAs - in this case, representative of names which only appeared in either 2015 or 2016, but did **not** appear in both years - we set **all=FALSE**. This only retains names which were used in both 2015 and 2016.

### We now create our **Total** column, a row sum of **Number_Of_Uses.x** and **Number_Of_Uses.y**. This column is indeed the Total number of times a baby name was chosen in the United States for 2015 and 2016.

final$Total <- rowSums(final[,c("Number_Of_Uses.x","Number_Of_Uses.y")])

## Final Data Cleanup

### Our client reveals critical information - he desires only girl names due to his preference of assigning only names selected for females to his now-revealed-to-be-female unborn child (daughter).
### As our client requests the top ten most popular "girl" names, we return that using the following:

final <- final[order(-final$Total),]

### First, we sort **final** by descending **Total** to bring the most popular names to the top.

girl <- head(final[final$Gender=="F",],10)

### Then we select only the first ten rows where the **Gender** is female, creating a new data frame, **girl**.

### We'll re-sort this data frame by descending **Total** and select only the **Name** and **Total** columns.

itsagirl <- girl[order(-girl$Total),c("Name","Total")]

### This creates the **itsagirl** dataframe of only the client-request columns.

## Outputting the Final File

write.csv(itsagirl, "itsagirl.csv", row.names=FALSE)

### Per the client's request, we have now created a final file of the ten most popular girl's names, as defined by the sum of times it was used in 2015 and 2016 in the United States. **itsagirl.csv** contains only the **Name** and **Total**.
