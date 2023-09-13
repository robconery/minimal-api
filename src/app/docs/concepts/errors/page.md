## Error Handling Overview

If you have an unexpected error in your code, then ASP.NET Core automatically tries to show you what went wrong:

```c
app.MapGet("/dobad", () => int.Parse("this is not an int"));
```

If you visit the `dobad` URL in a browser, you'll see this:

![image](https://user-images.githubusercontent.com/234688/129421056-51d1b7aa-ed3c-4cc3-8bc8-f3d2b0715f06.png)

And the following will appear in the terminal that you run `dotnet run` in:

![image](https://user-images.githubusercontent.com/234688/129421155-fb148b66-30ee-4156-ad5c-52c84f31ccac.png)

Both of these give you an idea of what's wrong. In this case, you tried to turn a string that isn't a number into an `int`.
