# Bell Curve between -1 and 1
`1 - smoothstep(0,1,abs(x))`
![](2022-05-14-23-49-08.png)

# Bell Curve between two points
![](2022-05-14-23-50-00.png)

# Smooth Stairstep
[Graphtoy Link](https://graphtoy.com/?f1(x,t)=0.5&v1=false&f2(x,t)=0.25&v2=false&f3(x,t)=x%20/%20f2(x)&v3=false&f4(x,t)=smoothstep(f1(x),%201,%20x)&v4=true&f5(x,t)=f2(x)%20*%20(floor(f3(x))%20+%20f4(f3(x)%20-%20floor(f3(x))%20))&v5=true&f6(x,t)=&v6=true&grid=1&coords=0.19007103391968105,0.1864844415925732,1.8199403492395938)

![](smoothstairs.png)

Based on a very good answer here:
https://math.stackexchange.com/questions/1671132/equation-for-a-smooth-staircase-function

> Let `s:[0,1]→[0,1]` be a smooth function representing a single step. Assume that there exists some `ϵ>0` such that `s(x)=0` for all `x<ϵ` and `s(x)=1` for all `x>1−ϵ`. Setting
> `f(x)=s(x−⌊x⌋)+⌊x⌋`
> then gives us a smooth staircase with steps of height and width 1. By rescaling f, we can get steps of arbitrary width w and height h:
> `f(h,w,x)=hf(x/w)=h(s(x/w−⌊x/w⌋)+⌊x/w⌋)`

# Parabola of desired width and height
![](2022-04-25-03-34-27.png)

# Sharp mountain like stuff
[Link](https://graphtoy.com/?f1(x,t)=abs(sin(x%20-%201))&v1=false&f2(x,t)=f1(-x)&v2=false&f3(x,t)=min(f1(x),%20f2(x))&v3=true&f4(x,t)=&v4=false&f5(x,t)=&v5=false&f6(x,t)=&v6=false&grid=1&coords=0,0,5.089171420469821)
![](2022-05-01-21-53-15.png)
![](2022-05-01-21-53-26.png)