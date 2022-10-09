This looks promising:
https://mathparser.org/


Port of LINQ's Dynamic Expressions to older .NET standard versions - this works - but I had trouble adding custom functions.
https://github.com/zzzprojects/System.Linq.Dynamic.Core

Drop the DLL to Unity project, and write code like this:
```csharp
using System.Linq.Dynamic.Core;
using System.Linq.Expressions;
using UnityEngine;

    ...

    ParameterExpression x = Expression.Parameter(typeof(int), "x");
    ParameterExpression y = Expression.Parameter(typeof(int), "y");
    LambdaExpression e = DynamicExpressionParser.ParseLambda(
        parameters: new [] { x, y },
        resultType: null,
        expression: "(x + y) * 2");
    
    var compiled = e.Compile();
    var result = compiled.DynamicInvoke(1, 2);
```

Looks interesting but says .NET Standard 3
https://github.com/dynamicexpresso/DynamicExpresso