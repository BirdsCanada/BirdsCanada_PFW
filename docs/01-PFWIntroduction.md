# Introduction {#Intro1}



## Project FeederWatch {#intro1.1}

Welcome to Project FeederWatch!

Since 1987, the Cornell Lab of Ornithology and Birds Canada (formerly Bird Studies Canada) have partnered on Project FeederWatch to mobilize thousands of citizen scientists across North America to count birds in their backyards over the winter. Specifically, FeederWatch is a winter-long (November-April) survey of birds that visit feeders at backyards, nature centers, community areas, and other locales. Participants periodically count the birds they see at their feeders and send their counts to [Project FeederWatch](https://feederwatch.org/). Decades of data provide a comprehensive look at continental wintertime populations of feeder birds over the late 20th and early 21st centuries—including some compelling stories of range expansions and contractions, populations in flux, and birds adapting to changing environments.

## Goal {#intro1.2}

Our goal of this online ‘handbook’ is to demonstrate how to use the R statistical programming language (https://www.r-project.org/) to import, clean, and explore raw Project FeederWatch (PFW) data through visualizations and summaries, and run various analytic procedures. We hope the contents will be useful to researchers interested in PFW data. While this handbook is focused on Canadian PFW data, it could easily be used for US data with minor changes to the R code examples. If you have suggestions for additional examples, please let us know by emailing dethier@birdscanada.org.

If you have specific questions about Project FeederWatch, please reach out to our Program Leads: 

> Canada: pfw@birdscanada.org
> U.S: feederwatch@cornell.edu

## Getting Involved {#intro1.3}

Project FeederWatch is supported almost entirely by its participants. The annual participation fees cover materials, staff support, web design, data analysis, and the year-end report (Winter Bird Highlights). Without the support of our participants, this project wouldn’t be possible.

As a program that engages participants across the US and Canada, we strive to ensure that Project FeederWatch is accessible and welcoming to every person. FeederWatch is conducted by people of all skill levels and backgrounds, including children, families, individuals, classrooms, retired persons, youth groups, nature centers, and bird clubs.

Please join the project for the country in which you reside: [Canada](https://www.birdscanada.org/you-can-help/project-feederwatch/) or [U.S](https://join.birds.cornell.edu/page/33514/donate/1?ea.tracking.id=WEB)

Thank you for your contribution and participation!

## Prerequisites {#intro1.4}

This book assumes that you have a basic understanding of R. Regardless of whether you are new to R or not, we highly recommend that you become familiar with ‘R for Data Science’ by Garrett Grolemund and Hadley Wickham (http://r4ds.had.co.nz/). Their book covers how to import, visualize, and summarize data in R using the tidyverse collection of R packages (https://www.tidyverse.org/). It also provides an invaluable framework for organizing your workflow to create clean, reproducible code (http://r4ds.had.co.nz/workflow-projects.html). We follow their lead by, wherever possible, using the tidyverse framework throughout this book.

## Acknowledgements {#intro1.5}

Project FeederWatch is a joint project of the [Cornell Lab of Ornithology](https://www.birds.cornell.edu/home/) and [Birds Canada](https://www.birdscanada.org/). Project FeederWatch is sponsored in the U.S. and Canada by [Wild Birds Unlimited](https://www.wbu.com/?utm_source=Cornell+Lab+eNews&utm_campaign=cb85f4fa3c-PFW+eNews%3A+project+reminders_COPY_01&utm_medium=email&utm_term=0_47588b5758-cb85f4fa3c-) and in Canada by [Armstrong Bird Food](https://armstrongbirdfood.com/?utm_source=Cornell%20Lab%20eNews&utm_campaign=cb85f4fa3c-PFW%20eNews%3A%20project%20reminders_COPY_01&utm_medium=email&utm_term=0_47588b5758-cb85f4fa3c-). Many people have contributed to the success of PFW including its founder Erica Dunn, who also designed its protocol.  Thank you to David Bonter and Emma Greig at Cornell and Kerrie Wilcox at Birds Canada for organizing and managing PFW over the years. Thanks to David Bonter, Emma Greig, Denis Lepage for managing the Feederwatch database. The text in this document was adapted from The Cornell Lab [Project FeederWatch](https://feederwatch.org/) webpage. Funding provided by Environment and Climate Change Canada made the publication of this online resource possible. Thank you!
