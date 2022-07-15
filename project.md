title: "SARS-CoV-2 Data Analysis"
layout: page
permalink: https:://ronnie-cv.github.io/ucsd-summer2022/project

Project Progress: Week of 7/10  
Ronnie Volman  
Turakhia Lab, UCSD

```{r}
#Package installation
require(ggplot2)
require(dplyr)
```

```{r}
#Loading information previously processed with Chronumental + file with clade information. 
chronum_output = read.delim("~/projects/chronumental/chronumental_dates_public-2022-03-11.metadata.tsv.gz.tsv")
chronum_input = read.delim("~/projects/chronumental/public-2022-03-11.metadata.tsv.gz")
clade_df = read.delim("~/projects/chronumental/clade1")
```

```{r}
#Filtered chronum_output to only contain relevant information about nodes
#Merged filtered chronum_output df and clade_df by columns containing node information
clade_result = merge(clade_df,
                     chronum_output %>% dplyr::filter(grepl("node", strain)), 
                     by.x = "clade_root_node", by.y = "strain")
```

```{r}
#Merged previous data frame with chronum_input to produce final data frame
final_result = merge(chronum_input %>% 
                   dplyr::group_by(pangolin_lineage) %>% 
                   dplyr::summarize(first_date = min(date)), #Determined first observed date of each clade
                 clade_result, 
                 by.x = "pangolin_lineage", by.y = "clade")
final_filter = final_result %>% 
  dplyr::filter(nchar(first_date, type = "chars") == 10) #Filtered out incomplete dates (e.g., "2020-5")
final_filter$days = with(final_filter, difftime(first_date, predicted_date, units = "days")) #Appended column containing time difference (in days) between first observed and first emerged dates of the clade.
final_filter = final_filter %>% 
  dplyr::filter(as.numeric(days) > 0) #Filtered out days that appeared as negative numbers
#strsplit("A.11", split = ".", fixed = T)[[1]][1]  
final_result = final_result %>% dplyr::mutate(clade_letter = strsplit(pangolin_lineage, split = ".", fixed = T)[[1]][1]) #Appended column with clade letter.
unique(final_result$clade_letter)
```
