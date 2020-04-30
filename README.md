
# EnvRtype: a tool for envirotyping analysis and genomic prediction considering reaction norms

Authorship: [Allogamous Plant Breeding Lab (University of São Paulo, ESALQ/USP, Brazil)](http://www.genetica.esalq.usp.br/en/lab/allogamous-plant-breeding-laboratory)

Manteiner: [Germano Costa Neto](https://github.com/gcostaneto)

[Simplified Tutorial (R script)](https://raw.githubusercontent.com/allogamous/EnvRtype/master/tutorial_script_R.R)






## Background

Environmental typing (envirotyping) has proven useful in identifying the non-genetic drivers of phenotypic adaptation in plant breeding. Combined with phenotyping and genotyping data, the use of envirotyping data may leverage the molecular breeding strategies to cope with environmental changing scenarios. Over the last ten years, this data has been incorporated in genomic-enabled prediction models aiming to better model genotype x environment interaction (GE) as a function of reaction norm. However, there is difficult for most breeders to deal with the interplay between envirotyping, ecophysiology, and genetics. Here we present the EnvRtype R package as a new toolkit developed to facilitate the interplay between envirotyping and genomic prediction. This package offers three modules: (1) collection and processing data set, (2) environmental characterization, (3) build of ecophysiological enriched genomic prediction models accounting for three different structures of reaction norm. Here we focus our efforts to present a practical use of EnvRtype package in supporting the genome-wide prediction of reaction norms. We provide a intuitive framework to integrate different reaction norm models in Bayesian Genomic Genotype x Environment Interaction (BGGE) package.

## **Features and Functionality**

EnvRtype consists of the following three modules, which collectively generate a simple workflow to collect, process and integrated envirotyping into genomic prediction in multiple environments.

- [Environmental Sensing Module (ES)](#heading1)
  * [Basic summary statistics for environmental data](#sub-heading)
- [Environmental Characterization Module (EC)](#heading)
  * [Environmental Typologies based on Cardinal Limits](#sub-heading)
- [Reaction Norm Module (RN)](#heading)


## Install
```{r}
library(devtools)
install_github('allogamous/EnvRtype')
require(EnvRtype)
```
<!-- toc -->

# Environmental Sensing Module (ES)
```{r}
lat = c(-13.05,-12.32,-18.34,-18.90,-23.03) # vector of latitude WGS84
lon = c(-56.05,-55.42,-46.31,-49.56,-51.02) # vector of lontitude WGS84
env = c("NM","SO","PM","IP","SE")           # vector of environment/site ID
plant.date = c("2015-02-15","2015-02-13", # vector of start period
               "2015-02-26","2015-03-01",
               "2015-02-19") 
harv.date =rep("2015-06-15",5) # vector of end period
```

- So we can use this information to collect weather data from NASAPOWER:
```{r}
df.clim <- get_weather(env.id = env,lat = lat,lon = lon,
                       start.day = plant.date,end.day = harv.date, asdataframe = F) # returns a list of dataframes by environments

df.clim <- get_weather(env.id = env,lat = lat,lon = lon,
                       start.day = plant.date,end.day = harv.date) # returns a dataframe with all environments by default
                       
head(df.clim)

```
- Basic processing of get_weather() 
```{r}
df.clim <-processWTH(x = df.clim,lon = 'LON',lat = 'LAT',env.id = 'env',download.ALT = TRUE,country = 'BRA')
```
## Basic summary statistics for environmental data
```{r}
summaryWTH(df.clim)
summaryWTH(df.clim,env.id = 'env')
```

- Summary a particular environmental variable
```{r}
summaryWTH(df.clim,env.id = 'env',var.id = 'T2M')
summaryWTH(df.clim,env.id = 'env',var.id = c('T2M','T2M_MAX')) # or more than one
```
- Summary by time intervals (e.g., phenology)
```{r}
summaryWTH(df.clim,env.id = 'env',by.interval = T)
```
- Summary by time intervals given by time.window
```{r}
summaryWTH(df.clim,env.id = 'env',by.interval = T,time.window = c(0,14,35,60,90,120))
```
- Summary by time intervals given by time.window and names.window
```{r}
summaryWTH(df.clim,env.id = 'env',by.interval = T,
           time.window = c(0,14,35,60,90,120),
           names.window = c('P-E','E-V1','V1-V4','V4-VT','VT-GF','GF-PM'))

```
- Returns only mean values
```{r}
summaryWTH(df.clim,env.id = 'env',statistic = 'mean')
```
- Returns only sum values
```{r}
summaryWTH(df.clim,env.id = 'env',statistic = 'sum')
```
- Returns quantile values (default = 25%, 50% and 75%)
```{r}
summaryWTH(df.clim,env.id = 'env',statistic = 'quantile')
```
- For specific quantiles (e.g., 20%, 76% and 90%)
```{r}
summaryWTH(df.clim,env.id = 'env',statistic = 'quantile',probs = c(.20,.76,.90))
```

## **Module II: Building Environmental Covariable Matrices**

- Mean-centered and scaled matrix
```{r}
W.matrix(df.cov = df.clim,by.interval = F)
```
- Same as summaryWTH, we can add time.windows
```{r}
W.matrix(df.cov = df.clim,by.interval = T,
         time.window = c(0,14,35,60,90,120))
```
- Select the statistic to be used
```{r}
W.matrix(df.cov = df.clim,by.interval = T,statistic = 'mean',
         time.window = c(0,14,35,60,90,120))
         

W.matrix(df.cov = df.clim,by.interval = T,statistic = 'quantile',
         time.window = c(0,14,35,60,90,120))

```

- We can perform a Quality Control (QC) based on the maximum sd tolered
```{r}
W.matrix(df.cov = df.clim,by.interval = F,QC = T)
```
- We can perform a Quality Control (QC) based on the maximum sd tolered
```{r}
W.matrix(df.cov = df.clim,by.interval = F,QC = T,sd.tol = 3)
```
- We can perform a Quality Control (QC) based on the maximum sd tolered
```{r}
W.matrix(df.cov = df.clim,by.interval = F,QC = T,sd.tol = 2)
```
- Create for specific variables
```{r}
id.var = c('T2M_MAX','T2M_MIN','T2M')
W.matrix(df.cov = df.clim,var.id = id.var)
```
- Or even combine with summaryWTH by using is.processed=T
```{r}
data<-summaryWTH(df.clim,env.id = 'env',statistic = 'quantile')
W.matrix(df.cov = data,is.processed = T)
```
# Environmental Characterization Module (EC)

## Environmental Typologies based on Cardinal Limits

```{r}
EnvTyping(df.cov = df.clim,env.id = 'env',var.id='T2M')
```
- Typologies by.intervals (generic time intervals)
```{r}
EnvTyping(df.cov = df.clim,env.id = 'env',var.id='T2M',by.interval = T)
```
- Typologies by.intervals (specific time intervals)
```{r}
EnvTyping(df.cov = df.clim,env.id = 'env',var.id='T2M',by.interval = T,time.window = c(0,15,35,65,90,120))
```
- Typologies by.intervals (specific time intervals and with specific names)
```{r}
names.window = c('1-intial growing','2-leaf expansion I','3-leaf expansion II',
                 '4-flowering','5-grain filling','6-maturation')
out<-EnvTyping(df.cov = df.clim,env.id = 'env',var.id='T2M',by.interval = T,
               time.window = c(0,15,35,65,90,120),
               names.window = names.window)
```
- OBS: some possible plots with ggplot2....
```{r}
# plot 1: enviromental variables panel
require(ggplot2)
ggplot() + 
  scale_x_discrete(expand = c(0,0))+
  scale_y_discrete(expand = c(0,0))+
  scale_fill_gradientn(colours= rainbow(15))+
  geom_tile(data=out,aes(x=reorder(env,Freq), y=reorder(env.variable,Freq),fill=Freq))+
  ylab('Envirotype ID\n')+ 
  xlab("\nEnvironment ID")+
  labs(fill='Frequency')+
  theme(axis.title = element_text(size=19),
        legend.text = element_text(size=9),
        strip.text.y  = element_text(size=13,angle=360),
        legend.title = element_text(size=17),
        strip.background = element_rect(fill="gray95",size=1),
        legend.position = 'bottom')

# plot 2: distribution of envirotypes
ggplot() + 
  #theme_void()+
  scale_x_discrete(expand = c(0,0))+
  scale_y_continuous(expand = c(0,0))+
  # theme_pubclean()+
  #scale_fill_manual(values=c('red2','orange2','green3','blue3','violet'))+
  facet_grid(var~interval)+
  geom_bar(data=out, aes(y=Freq, x=env,fill=env.variable), 
           position = "fill",stat = "identity",width = 1,size=.2)+
  # scale_y_continuous(labels = scales::percent,expand = c(0,0))+ #coord_flip()+
  ylab('Absolute Frequency\n of Occurence\n')+ 
  xlab("\nEnvironment ID")+
  labs(fill='Envirotype')+
  theme(axis.title = element_text(size=19),
        #   axis.text = element_blank(),
        legend.text = element_text(size=9),
        strip.text = element_text(size=17),
        legend.title = element_text(size=17),
        strip.background = element_rect(fill="gray95",size=1),
        legend.position = 'bottom')

```
- For more than one variable, we can use the quantiles for all environments
```{r}
EnvTyping(df.cov = df.clim,var.id =  c('T2M','PRECTOT','WS2M'),env.id='env',by.interval = T)
```
- We can define the cardinals for each variable
```{r}
(cardinals= list(T2M=c(0,9,22,32,45),PRECTOT=c(0,5,10),WS2M=c(0,1,5)))

EnvTyping(df.cov = df.clim,var.id =  c('T2M','PRECTOT','WS2M'),
          cardinals = cardinals,env.id='env')

```
- However, we do not always have ecophysiological information about the best possible cardinals ... so we use quantiles!
If quantiles = NULL, 1%, 25%, 50%, 99% is assumed
```{r}
(cardinals= list(T2M=c(0,9,22,32,45),PRECTOT=c(0,5,10),WS2M=NULL))
EnvTyping(df.cov = df.clim,var.id =  c('T2M','PRECTOT','WS2M'),
          cardinals = cardinals,env.id='env')
```
- All analyses can also be run considering centered on the mean and scaled x ~ N (0.1)
```{r}
EnvTyping(df.cov = df.clim,var.id = 'PRECTOT',env.id='env',scale = T)
EnvTyping(df.cov = df.clim,var.id =  c('T2M','PRECTOT','WS2M'),env.id='env',scale = T) 
```

# Reaction Norm Module (RN)

We provide Genomic and Envirotypic kernels for reaction norm prediction. After generate the kernels, the user must use the [BGGE](https://github.com/italo-granato/BGGE) package to run the models

- Toy Example: genomic prediction for grain yield in tropical maize
```{r}
data("maizeYield") # 150 maize hybrids over 5 environments (grain yield data)
data("maizeG")     # GRM for maizeYield
data('maizeWTH')   # weather data for maize Yield

Y <- maizeYield
G <- maizeG
df.clim <- maizeWTH

```
- Returns benchmark main effect model: 

<p align="center">
  <img width="120" height="18" src="/fig/mod1.png">
</p>

```{r}
MM <- get_kernel(K_G = list(G=G),Y = Y,reaction = F,model = 'MM')
```
- Returns benchmark main GxE deviation model:

<p align="center">
  <img width="160" height="18" src="/fig/mod2.png">
</p>

```{r}
MDs <-get_kernel(K_G = list(G=G),Y = Y,reaction = F,model = 'MDs')
```
- Obtaining environmental variables based on quantiles

```{r}
W.cov<-W.matrix(df.cov = df.clim,by.interval = T,statistic = 'quantile',
                time.window = c(0,14,35,60,90,120))
dim(W.cov)

```
- Creating Env Kernels from W matrix and Y dataset

```{r}
H <- EnvKernel(df.cov = W.cov,Y = Y,merge = T,env.id = 'env')
dim(H)
dim(H$varCov) # variable relationship
dim(H$envCov) # environmental relationship

#env.plots(H$envCov,row.dendrogram = T,col.dendrogram = T) # superheat
superheat(H$envCov,row.dendrogram = T,col.dendrogram = T)

```
- Parametrization by 

<p align="center">
  <img width="110" height="50" src="/fig/mod3.png">
</p>

```{r}
H <- EnvKernel(df.cov = W.cov,Y = Y,merge = T,env.id = 'env',bydiag=FALSE)
dim(H)
dim(H$varCov) # variable relationship
dim(H$envCov) # environmental relationship

#env.plots(H$envCov,row.dendrogram = T,col.dendrogram = T) # superheat
superheat(H$envCov,row.dendrogram = T,col.dendrogram = T)

```
- Parametrization by 

<p align="center">
  <img width="130" height="50" src="/fig/mod4.png">
</p>

resulting in diag(K_W) = 1

```{r}
H <- EnvKernel(df.cov = W.cov,Y = Y,merge = T,env.id = 'env',bydiag=TRUE)
dim(H)
dim(H$varCov) # variable relationship
dim(H$envCov) # environmental relationship

#env.plots(H$envCov,row.dendrogram = T,col.dendrogram = T) # superheat
superheat(H$envCov,row.dendrogram = T,col.dendrogram = T)

```
- Gaussian parametrization by 

<p align="center">
  <img width="130" height="50" src="/fig/mod5.png">
</p>

which d = dist(W), q = median(d) and h = gaussian parameter (default = 1)

```{r}
H <- EnvKernel(df.cov = W.cov,Y = Y,merge = T,env.id = 'env',gaussian=TRUE)
dim(H)
dim(H$varCov) # variable relationship
dim(H$envCov) # environmental relationship

#env.plots(H$envCov,row.dendrogram = T,col.dendrogram = T) # superheat
superheat(H$envCov,row.dendrogram = T,col.dendrogram = T)

```
**________________________________________________________________________________________________________**  

**Attention**:\
K_G = list of genomic kernels;\
K_E = list of environmental kernels;\
reaction = TRUE, build the haddamard's product between genomic and envirotype-based kernels;\
reaction = FALSE, but K_E != NULL, only random environmental effects using K_E are incorporated in the model  

**________________________________________________________________________________________________________**  

- Returns benchmark main effect model plus random environmental covariables:

<p align="center">
  <img width="140" height="18" src="/fig/mod6.png">
</p>

```{r}
EMM <-get_kernel(K_G = list(G=G),K_E = list(W=H$envCov), Y = Y,model = 'EMM') 
```
- Returns benchmark main GxE deviation model plus random environmental covariables: 

<p align="center">
  <img width="180" height="18" src="/fig/mod7.png">
</p>

```{r}
EMDs <-get_kernel(K_G = list(G=G),Y = Y,K_E = list(W=H$envCov),model = 'EMDs') # or model = MDs

```
- Returns reaction norm model: 

<p align="center">
  <img width="200" height="18" src="/fig/mod8.png">
</p>

```{r}
RN <-get_kernel(K_G = list(G=G),K_E = list(W=H$envCov), Y = Y,model = 'RNMM')

```

- Returns a full reaction norm model with GE and GW kernels: 

<p align="center">
  <img width="220" height="18" src="/fig/mod9.png">
</p>

```{r}
fullRN <-get_kernel(K_G = list(G=G),K_E = list(W=H$envCov), Y = Y,model = 'RNMDs')

```

- Advanced options: **lets build again the W matrix**

```{r}
W.cov<-W.matrix(df.cov = df.clim,by.interval = T,statistic = 'quantile',
                time.window = c(0,14,35,60,90,120))

W <- EnvKernel(df.cov = W.cov,Y = Y,merge = T,env.id = 'env',bydiag=TRUE)

# by using size_E = 'environment', get_kernel directly takes a W of q x q environments and builds a n x n matrix as EnvKernel()
EMM <-get_kernel(K_G = list(G=G),K_E = list(W=W$envCov), Y = Y,,model = 'EMM',size_E = 'environment')


# Its possible to integrate more than one environmental kernel
T.cov<- EnvTyping(df.cov=df.clim,var.id =  c('T2M','PRECTOT','WS2M'),env.id='env',format = 'wide')
eT <- EnvKernel(df.cov =T.cov,Y = Y,merge = T,env.id = 'env',bydiag=TRUE)


EMM <-get_kernel(K_G = list(G=G),K_E = list(W=W$envCov,eT=eT$envCov), Y = Y,model = 'EMM',size_E = 'environment')
EMM$KE_W # kernel from W
EMM$KE_eT # kernel from T (envirotype)

```

- Integration with **BGGE package**

```{r}
require(BGGE)

 ne <- as.vector(table(maizeYield$env))
      fit <- BGGE(y = maizeYield$value,
                  K = EMM,
                  ne = ne,
                  ite = 1000,
                  burn = 100,
                  thin = 2,
                  verbose = TRUE)
```
<img align="right" width="110" height="100" src="/fig/logo_alogamas.png">
