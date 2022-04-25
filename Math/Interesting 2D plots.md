# Smooth Stairstep

![](smoothstairs.png)
Based on a very good answer here:
https://math.stackexchange.com/questions/1671132/equation-for-a-smooth-staircase-function

> Let `s:[0,1]→[0,1]` be a smooth function representing a single step. Assume that there exists some `ϵ>0` such that `s(x)=0` for all `x<ϵ` and `s(x)=1` for all `x>1−ϵ`. Setting
> `f(x)=s(x−⌊x⌋)+⌊x⌋`
> then gives us a smooth staircase with steps of height and width 1. By rescaling f, we can get steps of arbitrary width w and height h:
> `f(h,w,x)=hf(x/w)=h(s(x/w−⌊x/w⌋)+⌊x/w⌋)`

# Parabola of desired width and height
![](2022-04-25-03-34-27.png)
