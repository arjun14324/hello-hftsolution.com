
```{r messsage=FALSE, warning=FALSE, echo = FALSE}
#{{{
shinyUI(fluidPage(
    fluidRow(
        column(6, 
               renderPrint({
                   library(orderbook)
                   library(shiny)
                   filename <- system.file("extdata","sample.txt",
                                           package = "orderbook")
                   ob <- orderbook(file = filename)
                   ob <- read.orders(ob, 10000)
		   display(ob)
                   ob <<- ob
               })
               ),
        column(6,
               renderPlot({ plot(ob) })
               )
    ))
)
))
#}}}
```

<div class="MIfooter"><img


## entering a market order
##

{{{
shinyUI(fluidPage(
    
    fluidRow(
        column(6, numericInput("amount2", "Number of Shares", value = 100)),
        column(6, radioButtons("typeBS2", "Buy or Sell", choices = c("Buy" = 1,
                                                      "Sell" = 2), selected = 1))
    ),
    
     
    fluidRow(
        column(5, offset = 1,
               renderPrint({
                   filename <- system.file("extdata",
                                           "sample.txt",
                                           package = "orderbook")
                   ob <- orderbook(file = filename)
                   ob <- read.orders(ob, 10000)
                   display(ob)
               })
               ),
        column(5, offset = 1,
               renderPrint({
                   if (input$amount2 <= 0) {
                       cat("Enter number of shares")
                   } else {
                     orderType <- switch(input$typeBS2, 
                                         "1" = "BUY",
                                         "2" = "SELL")
                     filename <- system.file("extdata",
                                             "sample.txt",
                                             package = "orderbook")
                     ob <- orderbook(file = filename)
                     ob <- read.orders(ob, 10000)
                     ob <- market.order(ob, input$amount2, orderType)
                     display(ob)
                   }
               })
               ))
      )
    )
#}}
