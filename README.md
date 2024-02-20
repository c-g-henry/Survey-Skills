# Survey-Skills
Using R to weight data, run regressions, create toplines and crosstabs, and find margin of error.


#pulling in some sample data and checking it out
poll <- read.csv("sample_data.csv")
names(poll)
summary(poll)
str(poll)

check <- complete.cases(poll)
summary(check)

#following instructions here: https://sdaza.com/blog/2012/raking/ for raking weights using anesrake package

#i'm going to weight sex, age5, and race5. 

#checking out those fields
summary(poll$sex)
summary(poll$age5)
summary(poll$race5)

summary(poll)

#trying out setting fields from data to factors and setting names.
poll$sex <- as.factor(poll$sex)
poll$age5 <- as.factor(poll$age5)
poll$race5 <- as.factor(poll$race5)

str(poll$sex)
str(poll$age5)
str(poll$race5)
summary(poll$sex)
summary(poll$age5)
summary(poll$race5)

#looking at current relative frequencies of these tables using weights package
library(weights)

wpct(poll$sex)

wpct(poll$age5)

wpct(poll$race5)


#next step is setting targets in list. i'm using made up targets fairly close to the original frequencies.
sextarget <- c(.52, .48)
age5target <- c(.24, .26, .18, .17, .15)
race5target <- c(.75, .07, .08, .06, .04)

#trying out setting names on targets --- yes this is a necessary step, and these must match names from factor.
names(sextarget) <- c("1","2")
names(age5target) <- c("1", "2", "3", "4", "5")
names(race5target) <- c("1", "2", "3", "4", "5")


targets <- list(sextarget, age5target, race5target)


names(targets) <- c("sex", "age5", "race5")


poll$caseid <- 1:length(poll$age5)

#checking that targets add to 1
sum(sex)
sum(age5)
sum(race5)

#pulling in anesrake
library(anesrake)


#*i'm not sure what this is doing or what's being checked here** i think(?) it's checking that difference is > 5%
#anesrakefinder(targets, poll, choosemethod = "total")

#applying anesrake function. here's a description from the instructions:
  #The maximum weight value is five, weights greater than five will be truncated (cap = 5).
  #The total differences between population and sample have to be greater than 0.05 so that to include a variable (pctlim = .05).
  #The maximum number of variables included in the raking procedure is three (nlim = 3).
  #force1 = TRUE is forcing each variable to sum to 1. 

#outsave <- anesrake(targets, poll, caseid = poll$caseid,
                    #verbose= FALSE, cap = 5, choosemethod = "total",
                   # type = "pctlim", pctlim = .05 , nlim = 3,
                   # iterate = TRUE , force1 = TRUE)

summary(sex)
str(sex)
summary(age5)
str(age5)
summary(race5)
str(race5)

#the levels don't have to be the same for this to work, checking them here.
levels(poll$sex)
levels(sextarget)

levels(poll$age5)
levels(age5target)

levels(poll$race5)
levels(race5target)

#running anesrake!!!
outsave <- anesrake(targets, poll, caseid=poll$caseid,
                    verbose=TRUE)


summary(outsave)

# add weights to the dataset
poll$weightvec  <- unlist(outsave[1])
n  <- length(poll$sex)

# weighting loss
((sum(poll$weightvec ^ 2) / (sum(poll$weightvec)) ^ 2) * n) - 1

#checking out weights
names(poll)

summary(poll$weightvec)
str(poll$weightvec)

write.csv(poll,"weighted_poll.csv")
