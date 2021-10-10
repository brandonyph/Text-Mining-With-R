Chapters 1. set up the path for both positive and negative folder 2. Set
up all keywords 3. Create a Word Count matrix of correct dimensions 4.
Loop through all pdf files and count the keywords of each files one by
one 5. Export the word Count as data frame 6. Using SVM for
classification 7. Using Neural Network for classification 8. Using
Logistic regression for classification 9. Test model on new data

# Setup Work Directory and List down all Pdf in the folder as a list object

``` r
setwd("D:/Users/Desktop/TextMining/")

PositiveFiles <- paste0("./Positive/",list.files(path = "./Positive",pattern = ".pdf"))
NegativeFiles <- paste0("./Negative/",list.files(path = "./Negative",pattern = ".pdf"))
files <- c(PositiveFiles,NegativeFiles)
```

# List all keywords you want to detect as another list object

``` r
keywords <-
  c(
    "taxonomy",
    "pcr",
    "amplification",
    "primer",
    "molecular",
    "cloud computing ",
    "phylogenomic",
    "genome",
    "illumina",
    "assembly",
    "species",
    "negative binomial",
    "molecular biology",
    "dna",
    "rna-seq",
    "degs",
    "sequencing",
    "population",
    "education",
    "participants"
    
  )
```

# Import Libarry

``` r
library(dplyr)
```

    ## 
    ## Attaching package: 'dplyr'

    ## The following objects are masked from 'package:stats':
    ## 
    ##     filter, lag

    ## The following objects are masked from 'package:base':
    ## 
    ##     intersect, setdiff, setequal, union

``` r
library(stringr)
library(pdftools)
```

    ## Using poppler version 21.04.0

# Create Intitalte Word Count matrix of correct dimenstions

``` r
filelength <- length(files)
wordlength <- length(keywords)
word_count <- seq(1,filelength*wordlength)
dim(word_count) <- c(filelength,wordlength)
```

# Loop through all pdf files and count the keywords of each files one by one

``` r
for (j in 1:length(files)) {
  P1 <- pdftools::pdf_text(pdf = files[j]) %>%
    str_to_lower() %>%
    str_replace_all("\\t", "") %>%
    str_replace_all("\n", " ") %>%
    str_replace_all("      ", " ") %>%
    str_replace_all("    ", " ") %>%
    str_replace_all("   ", " ") %>%
    str_replace_all("  ", " ") %>%
    str_replace_all("[:digit:]", "") %>%
    str_replace_all("[:punct:]", "") %>%
    str_trim()
  
  for (i in 1:length(keywords)) {
    word_count[j,i] <- P1 %>% str_count(keywords[i]) %>% sum()
  }
  
}
```

# Export the word Count as dataframe

``` r
word_matrix <- as.data.frame(word_count)

SeqPos <- paste0("P",seq(1,length(PositiveFiles)))
SeqNeg <- paste0("N",seq(1,length(NegativeFiles)))
SeqName <- c(SeqPos,SeqNeg)

rownames(word_matrix) <- SeqName
colnames(word_matrix) <- keywords

Label <- c(rep("Positive",length(PositiveFiles)),rep("Negative",length(NegativeFiles)))
Label
```

    ##  [1] "Positive" "Positive" "Positive" "Positive" "Positive" "Positive"
    ##  [7] "Positive" "Positive" "Positive" "Positive" "Positive" "Positive"
    ## [13] "Positive" "Positive" "Positive" "Positive" "Positive" "Positive"
    ## [19] "Positive" "Positive" "Positive" "Positive" "Positive" "Positive"
    ## [25] "Positive" "Positive" "Positive" "Positive" "Positive" "Positive"
    ## [31] "Positive" "Positive" "Positive" "Positive" "Positive" "Positive"
    ## [37] "Positive" "Positive" "Positive" "Positive" "Positive" "Positive"
    ## [43] "Positive" "Positive" "Positive" "Positive" "Positive" "Positive"
    ## [49] "Positive" "Positive" "Positive" "Positive" "Positive" "Positive"
    ## [55] "Positive" "Positive" "Positive" "Positive" "Positive" "Positive"
    ## [61] "Positive" "Negative" "Negative" "Negative" "Negative" "Negative"
    ## [67] "Negative" "Negative" "Negative" "Negative" "Negative" "Negative"
    ## [73] "Negative" "Negative" "Negative" "Negative" "Negative" "Negative"
    ## [79] "Negative" "Negative" "Negative" "Negative" "Negative" "Negative"
    ## [85] "Negative" "Negative" "Negative" "Negative" "Negative" "Negative"
    ## [91] "Negative" "Negative" "Negative" "Negative" "Negative" "Negative"
    ## [97] "Negative"

``` r
library(pheatmap)
pheatmap(word_matrix)
```

![](textmining_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

``` r
indices <- colSums(word_matrix) > 10

word_matrix2 <- word_matrix[,indices]
pheatmap(word_matrix2)
```

![](textmining_files/figure-gfm/unnamed-chunk-8-1.png)<!-- -->

# USing SVM for classification

``` r
library(e1071)

svm_x <- word_matrix2
svm_y <- as.factor(Label)
dat <- as.data.frame(cbind(svm_x,svm_y))

svm_model = svm(svm_y ~ ., data = dat, kernel = "linear", cost = 10, scale = FALSE)

svm_pred <- predict(svm_x, object = svm_model)

caret::confusionMatrix(svm_pred, svm_y)
```

    ## Confusion Matrix and Statistics
    ## 
    ##           Reference
    ## Prediction Negative Positive
    ##   Negative       33       17
    ##   Positive        3       44
    ##                                           
    ##                Accuracy : 0.7938          
    ##                  95% CI : (0.6997, 0.8693)
    ##     No Information Rate : 0.6289          
    ##     P-Value [Acc > NIR] : 0.0003538       
    ##                                           
    ##                   Kappa : 0.5909          
    ##                                           
    ##  Mcnemar's Test P-Value : 0.0036504       
    ##                                           
    ##             Sensitivity : 0.9167          
    ##             Specificity : 0.7213          
    ##          Pos Pred Value : 0.6600          
    ##          Neg Pred Value : 0.9362          
    ##              Prevalence : 0.3711          
    ##          Detection Rate : 0.3402          
    ##    Detection Prevalence : 0.5155          
    ##       Balanced Accuracy : 0.8190          
    ##                                           
    ##        'Positive' Class : Negative        
    ## 

``` r
library(glmnet)
```

    ## Loading required package: Matrix

    ## Loaded glmnet 4.1-2

``` r
glmmodel<-glm(svm_y~., family = binomial(), data=dat)
```

    ## Warning: glm.fit: fitted probabilities numerically 0 or 1 occurred

``` r
glmprop <- predict(svm_x,object=glmmodel)

glm.pred <- ifelse(glmprop > 0.5, "Positive", "Negative")

glm.pred <- as.factor(glm.pred)
glm_y <- as.factor(Label)

glm_cm <-caret::confusionMatrix(glm.pred, glm_y)

glm_cm$table
```

    ##           Reference
    ## Prediction Negative Positive
    ##   Negative       10        4
    ##   Positive       26       57

``` r
#https://www.statology.org/glm-fit-fitted-probabilities-numerically-0-or-1-occurred/
```

# Create Neural Network in Keras

``` r
###Create neural network
library(tensorflow)
library(keras)

copydata <- function(array,n)
    {
    for(i in 1:n){array <- rbind(array,array)}
    return(array)
}

train_data <- word_matrix2 %>% unlist()
dim(train_data) <- dim(word_matrix2)

train_data <- train_data %>% copydata(5)
# Convert labels to categorical one-hot encoding
y <- as.numeric(as.factor(Label)) -1 

train_label <- to_categorical(y,num_classes = 2)
train_label <- train_label %>% copydata(5)
```

``` r
model <- keras_model_sequential() %>% 
  layer_dense(units = 28, activation = "sigmoid", input_shape = c(ncol(train_data))) %>% 
  layer_dense(units = 56, activation = "sigmoid") %>% 
  layer_dense(units = 28, activation = "sigmoid") %>% 
  layer_dense(units = 14, activation = "sigmoid") %>%
  layer_dense(units = 2, activation = "softmax")

summary(model)
```

    ## Model: "sequential"
    ## ________________________________________________________________________________
    ## Layer (type)                        Output Shape                    Param #     
    ## ================================================================================
    ## dense_4 (Dense)                     (None, 28)                      504         
    ## ________________________________________________________________________________
    ## dense_3 (Dense)                     (None, 56)                      1624        
    ## ________________________________________________________________________________
    ## dense_2 (Dense)                     (None, 28)                      1596        
    ## ________________________________________________________________________________
    ## dense_1 (Dense)                     (None, 14)                      406         
    ## ________________________________________________________________________________
    ## dense (Dense)                       (None, 2)                       30          
    ## ================================================================================
    ## Total params: 4,160
    ## Trainable params: 4,160
    ## Non-trainable params: 0
    ## ________________________________________________________________________________

``` r
model %>% compile(
  optimizer = 'rmsprop',
  loss = 'categorical_crossentropy',
  metrics = c('accuracy')
)
```

# Train Neural Network

``` r
history <- model %>% 
  fit(
    x = train_data, y = train_label,
    epochs = 20,
    use_multiprocessing=TRUE,
    batch_size = 20
  )
```

# Testing Newly Created Neural Network on Testing data

``` r
TestFiles <- paste0("./Test/",list.files(path = "./Test",pattern = ".pdf"))
TestFiles
```

    ## [1] "./Test/Negative (1).pdf" "./Test/Negative (2).pdf"
    ## [3] "./Test/Negative (3).pdf" "./Test/Positive (1).pdf"
    ## [5] "./Test/Positive (2).pdf" "./Test/Positive (3).pdf"

``` r
filelength <- length(TestFiles)
wordlength <- length(keywords)

word_count_test <- matrix(nrow = filelength,ncol = wordlength)

for (j in 1:length(TestFiles)) {
  P1 <- pdftools::pdf_text(pdf = TestFiles[j]) %>%
    str_to_lower() %>%
    str_replace_all("\\t", "") %>%
    str_replace_all("\n", " ") %>%
    str_replace_all("      ", " ") %>%
    str_replace_all("    ", " ") %>%
    str_replace_all("   ", " ") %>%
    str_replace_all("  ", " ") %>%
    str_replace_all("[:digit:]", "") %>%
    str_replace_all("[:punct:]", "") %>%
    str_trim()
  
  for (i in 1:length(keywords)) {
    word_count_test[j,i] <- P1 %>% str_count(keywords[i]) %>% sum()
  }
  
}

word_matrix_test <- as.data.frame(word_count_test)

rownames(word_matrix_test) <- TestFiles
colnames(word_matrix_test) <- keywords

label_test <- c("Negative","Negative","Negative","Positive","Positive","Positive")
y <- as.factor(label_test)

x <- unlist(word_matrix_test[,indices])
dim(x) <- c(filelength,ncol(word_matrix2))
```

``` r
NN_prediction <- predict(model,x=x)
NN_prediction <- round(NN_prediction)

NN_outcome <- c()
for(i in 1:nrow(NN_prediction)){
  
  if(NN_prediction[i,1]){
  NN_outcome <- c(NN_outcome,"Negative")
  }else
  {NN_outcome <- c(NN_outcome,"Positive")
  }
}

NN_outcome <- as.factor(NN_outcome)

cm <- caret::confusionMatrix(NN_outcome, y)
cm$table
```

    ##           Reference
    ## Prediction Negative Positive
    ##   Negative        3        0
    ##   Positive        0        3

\`\`\`
