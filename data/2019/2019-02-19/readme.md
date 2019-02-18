# PhDs Awarded by Field

[Dr. Ellie Murray](https://twitter.com/EpiEllie) proposed a DataViz challenge for the `#epibookclub` based around the number of PhD degrees awraded in the USA. As an epidemiology postdoc she was especially interested in how others would approach DataViz for Epidemiology PhDs. There are additional fields within this dataset, so take a crack at whatever looks interesting!

The data comes from the [NSF](https://ncses.nsf.gov/pubs/nsf19301/data) - where there are at least 72 different datasets if you wanted to approach the data from a different angle. They are primarily summary tables stored as `.xlsx` files. Cleaning these can be a bit awkward so if you are interested, it would be a cool project to try to do fully in R!

Alternatively - I have cleaned the data for you and saved as a `.csv` for you to read directly into R. There was a lot of duplication, ie totals were represented within the broader table, and there wasn't a nice separation of the fields (simply indentation in the Excel sheet).

To get at the details for broad or major fields, `dplyr::summarize` is your friend!

```
phd_field <- readr::read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-02-19/phd_by_field.csv")
```

### Data Dictionary

[**`phd_by_field.csv`**](phd_by_field.csv)  

|variable    |class     |description |
|:-----------|:---------|:-----------|
|broad_field |character | The parent field (highest delineator)          |
|major_field |character | A sub-field below the broad field           |
|field       |character | The fine-field, most detailed          |
|year        |integer   | Year PhD awarded       |
|n_phds      |double    | Total number of PhDs awarded       |



### Spoilers - Cleaning Script

```
# Because all the things
library(tidyverse)
```

```
# Grab just the sub-major (major field) titles for separating the columns
sub_major_fields <- readxl::read_excel("sed17-sr-tab012.xlsx", skip = 3) %>% 
  rename(field = `Field of study`) %>% 
  filter(!is.na(field)) %>% 
  pull(field)

# Manually grabbed the broad fields (based off indentation)
major_fields <- c("Life sciences", 
                  "Physical Sciences and earth sciences", 
                  "Mathematics and computer sciences",
                  "Psychology and social sciences", 
                  "Engineering",
                  "Education",
                  "Humanities and arts",
                  "Other")

# read in Ellie's Dataset
# create new columns based off the matching of additional major or broad fields

df <- read_csv("https://raw.githubusercontent.com/eleanormurray/dataviz_challenges/master/sed17-sr-tab013.csv") %>% 
  mutate(Field = case_when(Field == "Othero" ~ "Other",
                           TRUE ~ Field),
         sub_major_field = case_when(Field %in% major_studies ~ Field,
                                 TRUE ~ NA_character_),
         major_field = case_when(Field %in% major_fields ~ Field,
                                 TRUE ~ NA_character_))

# Use tidyr::fill() to fill in the repeats of each major/broad field
df_field <- df %>% 
  fill(major_field, .direction = "down") %>% 
  fill(sub_major_field, .direction = "down") %>% 
  filter(!Field %in% major_fields) %>% 
  filter(!Field %in% major_studies)

# gather the years, remove the commas, and rename to appropriate columns
df_clean <- df_field %>% 
  gather(year, n_phds, `2008`:`2017`) %>% 
  mutate(year = factor(year),
         n_phds = parse_number(n_phds)) %>%
  rename(field = Field) %>% 
  select(broad_field = major_field, major_field = sub_major_field, field, year, n_phds)
  
```


```{r}

# Check to confirm numbers match (they do!)
df_clean %>% 
  group_by(major_field, year) %>% 
  summarize(sum(n_phds, na.rm = TRUE))
```


```{r}
# Write to .csv for posting
df_clean  %>% 
  write_csv("phd_by_field.csv")
```