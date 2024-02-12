## Using Dependency Injection

If you're coming to ASP.NET from other frameworks such as Node/Express, Ruby on Rails, or Django, you might be confused by documentation that shows services appearing out of nowhere. It takes some getting used to, but Dependency Injection and Inversion of Control give you a lot of flexibility, especially when it comes to testing.

The idea is straightforward: use an interface and pass it to the class that needs it, rather than that class creating an instance on its own.

Consider:

```csharp
public class ShoppingService{
  public Cart Cart {get; set;} = new Cart();
  public PaymentProcessor Processor {get; set;} = new PaymentProcessor();
  public AddItem(string sku){
    //if the cart has the item, increment it, other wise add it
  }
  public RemoveItem(string sku){
    //find the product and remove
  }
}
```

This is simplified code, of course, but highlights the principle of *coupling* very well. Our class can't exist without a direct dependency on `Cart` and `PaymentProcessor`, which means if I change something in either of those classes, my `ShoppingService` could easily break.

To loosen things up, we need to get to the core of what we need our `Cart` to do, which is to remember what our shopper wants, incrementing quantity as we go along, without duplicating items.

For that, we could create an interface which, ideally, isn't bound to the idea of a cart:

```csharp
public interface IShoppingList{
  public IDictionary<Product,int> Items {get; set;}
}
```

When creating interfaces, it's a good idea to focus on a single *ability*, rather than a concept. `IEnumerable` and `IQueryable` denote specific things in C#, which is what we want to do for our codebase.

Let's do another one for our `PaymentProcessor`:

```csharp
public interface IChargeable{
  public string Charge(IQuantifiedList items);
}
public interface IRefundable{
  public string Refund(string orderNumber);
}
```

Here, I've separated the ideas of a `Charge` and `Refund`, because we won't be handling refunds during the shopping process. That's an administrative thing! We can apply both of these to our processor class, however:

```csharp
public class Stripe: IChargeable, IRefundable{
  public string Charge(Cart cart){
    //...
  }
  public string Refund(string orderNumber){
    //...
  }
}
```

OK, let's get back to the `ShoppingService` and see why we did all of this:

```csharp
public class ShoppingService{
  public IShoppingList Cart {get; set;};
  public IChargeable Processor {get; set;};
  public ShoppingService(IShoppingList cart, IChargeable processor){
    Cart = cart;
    Processor = processor;
  }
  public AddItem(string sku, int quantity=1){
    //if the cart has the item, increment it, other wise add it
  }
  public RemoveItem(string sku){
    //find the product and remove
  }
}
```

We've decreased coupling dramatically, which means we now test our `ShoppingService` in a much easier way using a simple test double (aka "mock"):

```csharp
public class SuccessTestProcessor: IChargeable{
  string _code;
  public SuccessTestProcessor(string returnCode){
    _code = returnCode;
  }
  public string Charge(IShoppingList cart){
    return _code; //a successful charge
  }
}
public class ShoppingTests{
  IChargeable _processor;
  public ShoppingTests(){
    _processor = new SuccessTestProcessor("12324")
  }
  [Fact]
  public void A_cart_with_valid_items_creates_an_order_with_successful_charge(){
    //arrange the cart however you need for the test. We can just use the model here
    //since no data access
    var cart = new Cart();
    //send it in
    var svc = new ShoppingService(cart, _processor);
    //this won't use Stripe, just our cart
    var invoice = svc.Checkout();
  }
}
```

It might seem like we're jumping through a lot of hoops to loosen up our code and write more comprehensive tests, and that's true! It's not at all common to involve your database when running tests, unlike other frameworks and platforms, so patterns like this are needed.

## Using Inversion of Control

Dependency injection is neat, until it comes to actually using your application, which is when you find yourself creating a mess of instances and having to orchestrate them together like a musical conductor. This is where Inversion of Control comes in, which is typically a special library whose sole job is to do just that: *orchestrate your dependencies*.

To see this in action, let's add one more interface for our `ShoppingService`:

```csharp
public interface IShoppingService{
  public IShoppingList Cart {get;set;}
  public IChargeable Processor {get;set;}
  public string AddItem(string sku, int quantity);
  public string RemoveItem(string sku);
}
```

By creating this interface, we can leave it to our Inversion of Control container (`builder.Services`) to create it for us, based on the mapping we create in `Program.cs`. Let's do that now:

```csharp
var builder = WebApplication.CreateBuilder(args);
//...
builder.Services.AddScoped<IChargeable, Stripe>();
builder.Services.AddScoped<IRefundable, Stripe>();
builder.Services.AddScoped<IShoppingList, Cart>();
builder.Services.AddScoped<IShoppingService, ShoppingService>();
```

Our Inversion of Control container is smart enough to see the required dependencies and inject them where needed. As you can see, we're using `Stripe` to charge and refund things, and we're using our `Cart` model directly. When the container tries to add the `IShoppingService`, it will see that the constructure requires an `IShoppingList` as well as `IChargeable`, so it will grab the scoped instance and inject it for us. How nice.

Now, in our routes, we can ask for these services directly:

```csharp
app.MapPost("/cart/add-item", ([FromBody] CartRequest req,  [FromServices] IShoppingService shopping) => {
  shopping.AddItem(req.sku);
  //...
  return "OK";
})
```

Inversion of Control is a powerful thing. While it might seem verbose when you're getting started, you'll be quite happy with the "future-proofing" work you've done here as your application matures.