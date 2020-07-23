Why are you doing t-tests??
================

## I heard you like t-tests

Seems reasonable. It’s like the first test you learn in any stat class.
It’s foundational. A real classic test. I get that.

There’s good reasons to know what a t-test is and how it works.
Constructing a test statistic, comparing to a null, interpreting the
result – all that stuff is critical to understand, and a simple (one or
two sample) t-test is a great way to teach and learn that. Fine.

But – and you should learn this right after you learn what a t-test is –
you can do the actual test using regression and get exactly the same
results. And you can do all sorts of other tests using regression, too.
You can account for covariates. You can look at binary OR continuous
predictors. You can have categorical predictors with more than one
level. Slap “generalized” on your linear models and you can have binary
outcomes. Put a “mixed” in there are you can get all the repeated
measurements you want.

If you want all that with t-tests … get ready for a bunch of stupid
formulas that maybe you have to memorize for a stat quiz and never use
again.

Maybe you think someone taught you the formula for pooling variances
when you have unequal sample sizes for a reason. Maybe you’re thrilled
that you remember the difference between the usual test and Welch’s
test. Maybe you think you’re gonna need to know how an ANOVA table works
someday. t.test(extra \~ group, data = sleep)

Let me tell you – you learned that shit cause it mattered before they
invented regression and computers, and now it’s just there in your brain
for no reason. Your teacher thought “I had to learn this – better keep
teaching it”. Or worse “I got these slides from the last person, who got
them from the last person, who got them from someone, and well … it’s
useless but I’m busy / lazy.”

Okay, so maybe you think “I’ll just use t-tests for simple things, like
one and two sample tests, and regression for everything else.” And I
guess okay, if you want to remember one test for one case that you could
just as easily handle using the tool you use for ***everything else***,
then whatever. I can’t help you.

## I heard you like `t.test`

So, alright, maybe you think you are happy using t-tests for one or two
sample tests. And by the powers vested in you, you’re gonna use `t.test`
to do it. Here’s why that’s stupid.

Let’s use an example from the `t.test` help page.

``` r
with(sleep, t.test(extra[group == 1], extra[group == 2]))
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  extra[group == 1] and extra[group == 2]
    ## t = -1.8608, df = 17.776, p-value = 0.07939
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -3.3654832  0.2054832
    ## sample estimates:
    ## mean of x mean of y 
    ##      0.75      2.33

This is pulling two vectors out of `sleep` and doing a test on them. I’m
gonna leave aside that using `with` is something you shouldn’t do and
focus on using `t.test` with vectors. More often you’d do something
like:

``` r
grp_1 = sleep$extra[sleep$group == 1]
grp_2 = sleep$extra[sleep$group == 2]

t.test(grp_1, grp_2)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  grp_1 and grp_2
    ## t = -1.8608, df = 17.776, p-value = 0.07939
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -3.3654832  0.2054832
    ## sample estimates:
    ## mean of x mean of y 
    ##      0.75      2.33

There’s a lot about that code that I hate. The code to pull out the
vectors is gross, and the fact that you had to create vectors is stupid,
but … well, `t.test` likes vectors. A lot of base R stuff is just gross
code to create vectors that you don’t remember creating. And it works,
right? As long as you’re willing to forget about dataframes, and just
create some vectors … sure.

But `t.test` has a `data` argument, so maybe we can use dataframes like
civilized people. I’m gonna create a dataframe – based on the vector
inputs, *YOU MIGHT THINK* that `t.test` wants you to put the columns you
want to test side by side, so that’s what I’m gonna do.

``` r
new_df = 
  tibble(
    x = sleep$extra[sleep$group == 1],
    y = sleep$extra[sleep$group == 2]
  )
```

Spoiler alert: this doesn’t work.

``` r
t.test(x, y, data = new_df)
```

    ## Error in t.test(x, y, data = new_df): object 'x' not found

Maybe it needs quotes around variable names?

``` r
t.test("x", "y", data = new_df)
```

    ## Error in t.test.default("x", "y", data = new_df): not enough 'x' observations

Well that didn’t work … and from the error message I don’t know why at
all.

**IT TURNS OUT** that if you use `data`, you have to use a different
input structure. In fact, it’s something like this (also from the help
page):

``` r
t.test(extra ~ group, data = sleep)
```

    ## 
    ##  Welch Two Sample t-test
    ## 
    ## data:  extra by group
    ## t = -1.8608, df = 17.776, p-value = 0.07939
    ## alternative hypothesis: true difference in means is not equal to 0
    ## 95 percent confidence interval:
    ##  -3.3654832  0.2054832
    ## sample estimates:
    ## mean in group 1 mean in group 2 
    ##            0.75            2.33

So your choices with `t.test` are: (1) create some vectors and put them
in, or (2) put things in a dataframe (in way you might not expect) and
use a formula.

Not just any formula. The same formula you’d use if you were just going
to fit a damn linear model. In fact, check it out – this gives you the
same result:

``` r
lm(extra ~ group, data = sleep) %>% broom::tidy()
```

    ## # A tibble: 2 x 5
    ##   term        estimate std.error statistic p.value
    ##   <chr>          <dbl>     <dbl>     <dbl>   <dbl>
    ## 1 (Intercept)    0.750     0.600      1.25  0.228 
    ## 2 group2         1.58      0.849      1.86  0.0792

That’s right – same group means, same difference between groups, same
p-value. It’s the same.

There’s no reason to use `t.test` at all: you could literally just use
the same `lm` call you use for any other analysis.

## Okay so … now what

I’m assuming that you’re already doing a lot of things right. You’re
using dataframes. You don’t just create vectors and leave them laying
around. You know how to fit models using `lm` – even simple models, with
an intercept only or a binary predictor. You know how to interpret those
results.

So just … keep doing that.

And the next time you have to run a t-test for a collaborator, say
“okay” and just do that using regression.
