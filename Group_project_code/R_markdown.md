---
title: "STA 141 A project 2"
author: "Jason Xie"
date: "2021/2/13"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

Data_Analysis
```{r}
library(ggplot2)
library(tidyverse)
library(readr)
WHO <-  read.csv("https://covid19.who.int/WHO-COVID-19-global-data.csv")

population <- read.csv('https://raw.githubusercontent.com/YixinXie10086/Project-141A/master/Main%20project/population_by_country_2020%20(1).csv') %>% filter_all(all_vars(!is.na(.)&(.!="N.A.")))

head(WHO)
head(population)
```

```{r}
#Merge WHO data with Population data
names(population)[1] <- "Country"
names(WHO)[1] <- "date"
total_case=WHO %>% filter((date=='2021-02-18')) %>% mutate(death_percent=100*(Cumulative_deaths/Cumulative_cases))%>%
  filter(!is.na(death_percent)) %>% select(Country,Cumulative_cases,WHO_region)

head(total_case)
head(population)

data_attempt1=inner_join(total_case,population,by=c('Country'))
dim(data_attempt1)

#matching country_name in 2 different datasets
match_country_name=function(set1,name1,name2){
  for (val in c(1:length(name1))){
    set1$Country=as.character(set1$Country)
    set1$Country[set1$Country==name1[val]]=as.character(name2[val])
   
    set1$Country=as.factor(set1$Country)
  }
  
  return(set1)
}

name1=c('United States','Russia','Vietnam','Iran','United Kingdom',"Tanzania","South Korea","Venezuela","Côte d’Ivoire","North Korea","Syria","Bolivia","Czech Republic (Czechia)","Laos","Moldova","Brunei","Sao Tome & Principe",'Micronesia',"St. Vincent & Grenadines","Northern Mariana Islands","Saint Kitts & Nevis","Turks and Caicos","Wallis & Futuna","Saint Pierre & Miquelon",'Falkland Islands (Malvinas)')

name2=c("United States of America","Russian Federation","Viet Nam","Iran (Islamic Republic of)","The United Kingdom","United Republic of Tanzania","Republic of Korea","Venezuela (Bolivarian Republic of)","Côte d’Ivoire","Democratic People's Republic of Korea","Syrian Arab Republic","Bolivia (Plurinational State of)","Czechia","Lao People's Democratic Republic","Republic of Moldova","Brunei Darussalam","Sao Tome and Principe","Micronesia (Federated States of)","Saint Vincent and the Grenadines","Northern Mariana Islands (Commonwealth of the)","Saint Kitts and Nevis","Turks and Caicos Islands","Wallis and Futuna","Saint Pierre and Miquelon","Falkland Islands (Malvinas)")

population=match_country_name(population,name1,name2)






data_attempt2=inner_join(total_case,population,by=c('Country'))
data_attempt2
dim(data_attempt2)
```
```{r}
GDP_per_capita_19= read_csv('https://raw.githubusercontent.com/YixinXie10086/Project-141A/master/Main%20project/GDP%20per%20capita%20(2017%20PPP%20%24).csv',skip=5)%>% select(Country,`2019`)

colnames(GDP_per_capita_19)[2]="GDP per Capita $PPP 2019"

GDP_per_capita_19$Country=as.character(GDP_per_capita_19$Country)


list1=c("Tanzania (United Republic of)",'United Kingdom','United States',"Korea (Republic of)","Moldova (Republic of)")
list2=c("United Republic of Tanzania","The United Kingdom","United States of America","Republic of Korea","Republic of Moldova")

GDP_per_captia_19=match_country_name(GDP_per_capita_19,list1,list2)
data3=inner_join(data_attempt2,GDP_per_captia_19)%>% filter(`GDP per Capita $PPP 2019`!="..") 
data3

dim(data3)

```








```{r}

health_expenditure <- read.csv("https://raw.githubusercontent.com/YixinXie10086/Project-141A/master/Main%20project/Current%20health%20expenditure%20(%25%20of%20GDP).csv",skip = 6, header = T) %>%
  filter(X2017!="..") %>% select(Country,X2017) 

colnames(health_expenditure)[2]="health expenditure in percent of GDP"

for (i in c(1:length(health_expenditure$Country))){
  health_expenditure$Country[i]=substr(health_expenditure$Country[i],2,nchar(health_expenditure$Country[i]))
}



health_expenditure=as.tibble(health_expenditure)



list1=c("Congo (Democratic Republic of the)","Côte d'Ivoire","Eswatini (Kingdom of)","Korea (Republic of)","Lao People's Democratic Republic","Moldova (Republic of)","United Kingdom","United States")
list2=c("Democratic Republic of the Congo","Côte d’Ivoire","Eswatini","Republic of Korea","Lao People's Democratic Republic","Republic of Moldova","The United Kingdom","United States of America")


health_expenditure2=match_country_name(health_expenditure,list1,list2)
health_expenditure2$Country=as.character(health_expenditure2$Country)
#health_expenditure2$Country=as.character(health_expenditure$Country)

data4=inner_join(data3,health_expenditure2,by="Country")
health_expenditure2$Country

data4
dim(data4)

d_rate_table<- data4 %>% select(-Country) %>% mutate_if(is.character,as.factor)  

d_rate_table$`Yearly.Change`=as.character(d_rate_table$`Yearly.Change`)
d_rate_table$`Urban.Pop..`=as.character(d_rate_table$`Urban.Pop..`)
d_rate_table$`World.Share`=as.character(d_rate_table$`World.Share`)

#remove the % sign and change features into numberic
for (i in c(1:length(d_rate_table$`Yearly.Change`))){
  d_rate_table$`Yearly.Change`[i]=substr(d_rate_table$`Yearly.Change`[i],1,nchar(d_rate_table$`Yearly.Change`[i])-1)
}


for (i in c(1:length(d_rate_table$`Urban.Pop..`))){
  d_rate_table$`Urban.Pop..`[i]=substr(d_rate_table$`Urban.Pop..`[i],1,nchar(d_rate_table$`Urban.Pop..`[i])-1)
}

for (i in c(1:length(d_rate_table$`World.Share`))){
  d_rate_table$`World.Share`[i]=substr(d_rate_table$`World.Share`[i],1,nchar(d_rate_table$`World.Share`[i])-1)
}


d_rate_table=d_rate_table %>%
  transmute(infected_rate=(Cumulative_cases/`Population..2020.`),WHO_region=WHO_region,Population2020=`Population..2020.`,Pop_yearly_change=as.numeric(`Yearly.Change`),Net.Change=Net.Change,population_density=`Density..P.KmÂ².`,land_area=`Land.Area..KmÂ².`,Migrant_net_change=`Migrants..net.`,Fert_rate=as.numeric(`Fert..Rate`), median_age=as.numeric(`Med..Age`),urban_population_percent=as.numeric(`Urban.Pop..`),GDP_per_capita=as.numeric(`GDP per Capita $PPP 2019`),health_expenditure_of_GDP=as.numeric(`health expenditure in percent of GDP`))



  
                                     

d_rate_table
dim(d_rate_table)
#d_rate_table is the dataframe to use 
```

Modeling. 
```{r}

#Lasso approach
set.seed(1)
library(caret)
library(glmnet)
set.seed(1)
d_rate_table=as.tibble(d_rate_table)
#partition data into 50 percent. 
index_80percent= createDataPartition(d_rate_table$infected_rate,p=0.8,list=FALSE,times=1)

#test and training.
train=d_rate_table[index_80percent,]
test=d_rate_table[-index_80percent,]


#train=d_rate_table %>% sample_frac(0.8)
#test=d_rate_table %>% setdiff(train)
x_train=model.matrix(infected_rate~.,train)[,-1]
x_test=model.matrix(infected_rate~.,test)[,-1]
y_train=train %>% select(infected_rate) %>% unlist() %>% as.numeric()
y_test=test %>% select(infected_rate) %>% unlist() %>% as.numeric()


lasso_mod=glmnet(x_train,y_train,alpha=1 )
plot(lasso_mod)
cv.train=cv.glmnet(x_train,y_train,alpha=1,type.measure = "mse", standardize = TRUE, nfolds = 10)
plot(cv.train)
best_lam=cv.train$lambda.min
best_lam 

lasso_pred = predict(lasso_mod, s = best_lam, newx = x_test)
mean((lasso_pred - y_test)^2)

r2 <- cv.train$glmnet.fit$dev.ratio[which(cv.train$glmnet.fit$lambda == cv.train$lambda.1se)] 
r2
Rmse_lasso=sqrt(cv.train$cvm[cv.train$lambda == cv.train$lambda.1se])
Rmse_lasso
lasso_coef = predict(lasso_mod, type = "coefficients", s = best_lam)
lasso_coef




#ridge regression 

ridge_mod=glmnet(x_train,y_train,alpha=0)
plot(lasso_mod)
cv.train=cv.glmnet(x_train,y_train,alpha=0,type.measure = "mse", standardize = TRUE, nfolds = 10)




plot(cv.train)
best_lam=cv.train$lambda.min

best_lam 

ridge_pred = predict(ridge_mod, s = best_lam, newx = x_test)

mean((ridge_pred - y_test)^2)
r2 <- cv.train$glmnet.fit$dev.ratio[which(cv.train$glmnet.fit$lambda == cv.train$lambda.1se)] 
r2
Rmse_ridge=sqrt(cv.train$cvm[cv.train$lambda == cv.train$lambda.1se])
Rmse_ridge

ridge_coef = predict(ridge_mod, type = "coefficients", s = best_lam)
ridge_coef

```





```
datasource:
http://hdr.undp.org/en/indicators/194506#
https://www.kaggle.com/tanuprabhu/population-by-country-2020
http://hdr.undp.org/en/indicators/194906